/**
 * Marketing KPI Aggregator (n8n Function)
 *
 * WHAT THIS DOES (for non-technical reviewers)
 * ---------------------------------------------------------
 * This node turns raw email-campaign rows (daily totals) into a tidy set of
 * business-friendly summaries for a monthly report and chat assistant:
 *
 * - Overall performance: one roll-up line so leaders can see the "headline" KPIs.
 * - Send timing analysis: which weekday + time-window performed best/worst,
 *   so marketers can choose better delivery slots next month.
 * - Message effectiveness (RTE): compares creative variants M1/M2/M3 to decide
 *   which content to keep, scale, or pause.
 * - Audience & channel view: how Cardiologists vs Neurologists respond and how
 *   RTE vs Newsletter compare—useful for targeting and budget allocation.
 * - Mix shares: what % of total sends went to each channel, audience, or message;
 *   this prevents misleading rate comparisons by showing actual volume behind them.
 * - Weekly trend: a quick week-1..week-5 view to spot early spikes or fatigue.
 *
 * We also attach “data sufficiency” flags so rankings ignore very small samples.
 * This avoids over-reacting to tiny volumes that are statistically noisy.
 *
 * INPUT  (items[].json each row):
 *   month, event_date, send_window, campaign_id,
 *   specialty ("cardiologist"|"neurologist"),
 *   channel ("RTE"|"newsletter"),
 *   message_id (e.g., "M1","M2","M3","NL4312"),
 *   country ("Austria"|"Switzerland" ...),
 *   day_of_week (1..5),
 *   sends, delivered, opens, clicks, unsubs (integers)
 *
 * OUTPUT (single item consumed by the LLM/email/chat step):
 * {
 *   meta: {...},                  // high-level context & thresholds
 *   overall: {...},               // headline KPIs for the filtered slice
 *   by_slot: [...],               // weekday × window KPIs (+ sufficiency flags)
 *   top_slots: [...],             // best 3 timing slots (enough sample size)
 *   worst_slots: [...],           // worst 3 timing slots (enough sample size)
 *   rte_messages: [...],          // M1/M2/M3: KPIs + deltas vs best + share of RTE volume
 *   by_specialty: [...],          // Cardiologist vs Neurologist KPIs
 *   by_specialty_channel: [...],  // Same split but per channel
 *   by_channel: [...],            // RTE vs Newsletter KPIs
 *   mix: {                        // “what we actually sent” context (shares)
 *     by_channel: [...],
 *     by_specialty: [...],
 *     rte_message_mix: [...]      // message split inside RTE only
 *   },
 *   trend: {
 *     by_week: [...]              // week-of-month movement (fatigue/learning)
 *   }
 * }
 */

// -------------------------------
// 1) Utilities & constants
// -------------------------------

/**
 * Why we set a minimum “delivered” to rank timing slots:
 * ------------------------------------------------------
 * Tiny samples look exciting but are unreliable. We exclude slots below this
 * cutoff from best/worst lists so recommendations don’t overfit small numbers.
 */
const MIN_DELIVERED_FOR_RANK = 100; // tweak in one place if your org prefers a different floor

/** Human labels for weekday numbers (for nicer tables in the report) */
const DOW_LABELS = {1:'Mon', 2:'Tue', 3:'Wed', 4:'Thu', 5:'Fri'};

/** Defensive numeric parsing to avoid NaNs from the source data */
function safeNum(v) { const n = Number(v); return Number.isFinite(n) ? n : 0; }

/** Rate helper: returns 0 if denominator is 0 (prevents divide-by-zero noise) */
function ratio(n, d) { return d > 0 ? (n / d) : 0; }

/**
 * Per-1k helper:
 * --------------
 * “Clicks per 1k delivered” and “Unsubs per 1k delivered” are executive-friendly:
 * they translate rates into round numbers that feel tangible across volumes.
 */
function perThousand(n, d) { return d > 0 ? (1000 * n / d) : 0; }

/**
 * Compute standard KPIs (and extras) from a pre-summed group:
 * - Delivery rate = Delivered / Sends (list quality + deliverability)
 * - Open rate     = Opens / Delivered (subject line + audience fit)
 * - Click rate    = Clicks / Delivered (end-to-end effectiveness)
 * - CTR (CTO)     = Clicks / Opens (message relevance after opening)
 * - Unsub rate    = Unsubs / Delivered (fatigue/risk guardrail)
 * - Per-1k metrics help execs compare across different base sizes
 */
function computeKpis(agg) {
  const sends     = safeNum(agg.sends);
  const delivered = safeNum(agg.delivered);
  const opens     = safeNum(agg.opens);
  const clicks    = safeNum(agg.clicks);
  const unsubs    = safeNum(agg.unsubs);

  const delivery_rate   = ratio(delivered, sends);
  const open_rate       = ratio(opens, delivered);
  const click_rate      = ratio(clicks, delivered);
  const ctr             = ratio(clicks, opens);
  const unsub_rate      = ratio(unsubs, delivered);

  const clicks_per_1k   = perThousand(clicks, delivered);
  const unsubs_per_1k   = perThousand(unsubs, delivered);

  return {
    sends, delivered, opens, clicks, unsubs,
    delivery_rate, open_rate, click_rate, ctr, unsub_rate,
    clicks_per_1k, unsubs_per_1k
  };
}

/** Sum aggregator used by groupings */
function addAgg(prev, row) {
  return {
    sends:     safeNum(prev.sends)     + safeNum(row.sends),
    delivered: safeNum(prev.delivered) + safeNum(row.delivered),
    opens:     safeNum(prev.opens)     + safeNum(row.opens),
    clicks:    safeNum(prev.clicks)    + safeNum(row.clicks),
    unsubs:    safeNum(prev.unsubs)    + safeNum(row.unsubs),
  };
}

/**
 * Generic “group by”:
 * We roll the granular rows up to the views marketers actually use to decide
 * timing, targeting, and creative. Each group keeps a simple sum of KPIs.
 */
function groupBy(rows, keyFn) {
  const m = new Map();
  for (const r of rows) {
    const k = keyFn(r);
    const prev = m.get(k) || { sends:0, delivered:0, opens:0, clicks:0, unsubs:0, __rows:[] };
    const next = addAgg(prev, r);
    next.__rows = prev.__rows; next.__rows.push(r);
    m.set(k, next);
  }
  return m;
}

/**
 * Week-of-month index (1..5):
 * Easy way to see if we front-load or if fatigue creeps in mid-month.
 * Example: 1–7 = week 1, 8–14 = week 2, etc.
 */
function weekIndex(dateISO) {
  const d = new Date(String(dateISO));
  const day = d.getUTCDate();
  return Math.floor((day - 1) / 7) + 1;
}

// -------------------------------
// 2) Normalize input rows
// -------------------------------
/**
 * We normalize labels and coerce numerics so the rest of the code can assume
 * clean inputs. This prevents case/spacing issues (e.g., "rte" vs "RTE").
 */
const rows = items.map(x => {
  const j = { ...x.json };

  // Standardize channel labels
  if (typeof j.channel === 'string') {
    const c = j.channel.trim().toLowerCase();
    j.channel = (c === 'rte') ? 'RTE' : (c === 'newsletter' ? 'newsletter' : j.channel);
  }

  // Force key numeric fields to numbers
  j.sends     = safeNum(j.sends);
  j.delivered = safeNum(j.delivered);
  j.opens     = safeNum(j.opens);
  j.clicks    = safeNum(j.clicks);
  j.unsubs    = safeNum(j.unsubs);

  return j;
});

// -------------------------------
// 3) Overall KPIs (headline view)
// -------------------------------
/**
 * This is the “one glance” card for managers:
 * - Are we reaching people (delivered)?
 * - Are they engaging (open/click)?
 * - Any risk signals (unsub rate)?
 */
const overallAgg = rows.reduce(addAgg, { sends:0, delivered:0, opens:0, clicks:0, unsubs:0 });
const overall = computeKpis(overallAgg);

// -------------------------------
// 4) Send timing analysis (weekday × time window)
// -------------------------------
/**
 * Why this matters:
 * Timing is a common “free” lever—no budget needed. If a slot (e.g., Tue 10:00)
 * outperforms consistently, we prioritize it. We also mark low-sample slots so
 * we don’t chase noise.
 */
const bySlot = [];
for (const [key, agg] of groupBy(rows, r => `${r.day_of_week}||${r.send_window}`).entries()) {
  const [dStr, win] = key.split('||');
  const d = Number(dStr);
  const kpi = computeKpis(agg);
  bySlot.push({
    day_of_week: d,
    day_label: DOW_LABELS[d] || String(d),
    send_window: win,
    sufficient_data: (kpi.delivered >= MIN_DELIVERED_FOR_RANK),
    ...kpi
  });
}

// Rank only slots with enough data; sort by Click Rate (then Open Rate)
const rankableSlots = bySlot.filter(x => x.sufficient_data);
rankableSlots.sort((a,b) => (b.click_rate - a.click_rate) || (b.open_rate - a.open_rate));
const topSlots = rankableSlots.slice(0, 3);
const worstSlots = [...rankableSlots].reverse().slice(0, 3);

// -------------------------------
// 5) RTE message effectiveness (creative testing)
// -------------------------------
/**
 * Why this matters:
 * Messages (M1/M2/M3) reflect different subject lines or content angles.
 * We compare them using Click Rate (primary) and Open Rate (secondary)
 * to recommend which creative to scale or pause.
 */
const rteRows = rows.filter(r => r.channel === 'RTE' && /^M\d+/i.test(String(r.message_id)));
const rte_messages = [];
if (rteRows.length) {
  // Within-RTE mix helps prevent “winner’s curse” on tiny volume.
  const rteTotalSends = rteRows.reduce((s,r) => s + safeNum(r.sends), 0);

  const msgMap = groupBy(rteRows, r => String(r.message_id).toUpperCase());
  for (const [msg, agg] of msgMap.entries()) {
    const kpi = computeKpis(agg);
    rte_messages.push({
      message_id: msg,
      share_of_rte_sends: rteTotalSends > 0 ? (kpi.sends / rteTotalSends) : 0,
      ...kpi
    });
  }
  // Best-to-worst by Click Rate, ties broken by Open Rate
  rte_messages.sort((a,b) => (b.click_rate - a.click_rate) || (b.open_rate - a.open_rate));
  // Delta vs best (signed percentage points) for clear “how far behind”
  const best = rte_messages[0];
  if (best) {
    for (const m of rte_messages) {
      m.delta_click_rate_pts = (m.click_rate - best.click_rate) * 100;
      m.delta_open_rate_pts  = (m.open_rate  - best.open_rate ) * 100;
    }
  }
}

// -------------------------------
// 6) Specialty & channel breakdown (targeting & budget)
// -------------------------------
/**
 * Why this matters:
 * Different audiences and channels behave differently. This split helps a manager
 * decide where to focus next (e.g., RTE for Neurologists).
 */
const by_specialty = [];
for (const [spec, agg] of groupBy(rows, r => String(r.specialty)).entries()) {
  by_specialty.push({ specialty: spec, ...computeKpis(agg) });
}

const by_specialty_channel = [];
for (const [key, agg] of groupBy(rows, r => `${r.specialty}||${r.channel}`).entries()) {
  const [specialty, channel] = key.split('||');
  by_specialty_channel.push({ specialty, channel, ...computeKpis(agg) });
}

const by_channel = [];
for (const [ch, agg] of groupBy(rows, r => String(r.channel)).entries()) {
  by_channel.push({ channel: ch, ...computeKpis(agg) });
}

// -------------------------------
// 7) Mix shares (what we actually sent)
// -------------------------------
/**
 * Why this matters:
 * Rates alone can mislead if one group has very little volume. Mix tables show
 * how the month’s sends were distributed across channels, audiences, and messages.
 */
const totalSends = overall.sends || 0;

const mix_by_channel = by_channel.map(r => ({
  channel: r.channel,
  sends: r.sends,
  share_of_sends: totalSends > 0 ? (r.sends / totalSends) : 0
}));

const mix_by_specialty = by_specialty.map(r => ({
  specialty: r.specialty,
  sends: r.sends,
  share_of_sends: totalSends > 0 ? (r.sends / totalSends) : 0
}));

const mix_rte_messages = (() => {
  const out = [];
  const onlyRteMsgs = rte_messages.filter(x => /^M\d+/.test(x.message_id));
  const rteSendTotal = onlyRteMsgs.reduce((s,m) => s + safeNum(m.sends), 0);
  for (const m of onlyRteMsgs) {
    out.push({
      message_id: m.message_id,
      sends: m.sends,
      share_within_rte: rteSendTotal > 0 ? (m.sends / rteSendTotal) : 0
    });
  }
  out.sort((a,b)=> (b.share_within_rte - a.share_within_rte));
  return out;
})();

// -------------------------------
// 8) Week-of-month trend (fatigue/learning)
// -------------------------------
/**
 * Why this matters:
 * Campaigns often launch heavy in week 1, then settle. This quick split can reveal
 * fatigue (declining opens/clicks) or improvements (better later weeks).
 */
const by_week_map = groupBy(
  rows.map(r => ({...r, __week: weekIndex(r.event_date)})),
  r => String(r.__week)
);
const trend_by_week = [];
for (const [wk, agg] of by_week_map.entries()) {
  trend_by_week.push({
    week_index: Number(wk), // 1..5
    ...computeKpis(agg)
  });
}
trend_by_week.sort((a,b)=> a.week_index - b.week_index);

// -------------------------------
// 9) Meta/context for traceability
// -------------------------------
/**
 * Meta includes what markets/months/audiences the slice covers and the
 * sufficiency threshold we used for rankings—so the email can disclose it.
 */
const meta = {
  rows_in: rows.length,
  countries: [...new Set(rows.map(r => r.country))],
  months:    [...new Set(rows.map(r => r.month))],
  specialties: [...new Set(rows.map(r => r.specialty))],
  channels:  [...new Set(rows.map(r => r.channel))],
  campaign_ids: [...new Set(rows.map(r => r.campaign_id))],
  thresholds: {
    min_delivered_for_ranking: MIN_DELIVERED_FOR_RANK
  }
};

// -------------------------------
// 10) Emit single structured JSON
// -------------------------------
return [{
  json: {
    meta,
    overall,
    by_slot: bySlot,
    top_slots: topSlots,
    worst_slots: worstSlots,
    rte_messages,
    by_specialty,
    by_specialty_channel,
    by_channel,
    mix: {
      by_channel: mix_by_channel,
      by_specialty: mix_by_specialty,
      rte_message_mix: mix_rte_messages
    },
    trend: {
      by_week: trend_by_week
    }
  }
}];
