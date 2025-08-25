<!doctype html>
<html lang="en">
<head>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width,initial-scale=1" />
<title>Marketing Insights Pretotype (n8n · Open-Source · Email + Chat)</title>
<style>
  :root{
    --fg:#111; --muted:#555; --rule:#e9ecef; --bg:#fff; --chip:#f6f7f8; --accent:#0b6efd;
  }
  html,body{margin:0;padding:0;background:var(--bg);color:var(--fg);font:16px/1.5 -apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,Arial,"Noto Sans",sans-serif;}
  .wrap{max-width:920px;margin:0 auto;padding:32px 20px 48px;}
  h1{font-size:28px;margin:0 0 8px;}
  h2{font-size:20px;margin:28px 0 8px;}
  h3{font-size:16px;margin:22px 0 6px;}
  p{margin:8px 0;}
  .lead{color:var(--muted);margin-bottom:16px}
  .pill{display:inline-block;background:var(--chip);border:1px solid var(--rule);border-radius:999px;padding:4px 10px;font-size:12px;margin:2px 6px 2px 0}
  .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(260px,1fr));gap:12px;margin:8px 0 16px}
  .card{border:1px solid var(--rule);border-radius:10px;padding:12px;background:#fff}
  .muted{color:var(--muted);font-size:14px}
  .hr{height:1px;background:var(--rule);margin:20px 0}
  a{color:var(--accent);text-decoration:none}
  a:hover{text-decoration:underline}
  ul{margin:8px 0 8px 20px}
  code,pre{background:#f8f9fa;border:1px solid var(--rule);border-radius:6px;padding:2px 6px}
  pre{padding:12px;overflow:auto}
  .kicker{font-size:12px;letter-spacing:.08em;text-transform:uppercase;color:var(--muted)}
  .callout{background:#fff;border:1px solid var(--rule);border-left:4px solid var(--accent);padding:10px 12px;border-radius:8px}
  .footer{color:var(--muted);font-size:12px;margin-top:28px}
</style>
</head>
<body>
  <div class="wrap">
    <h1>Marketing Insights Pretotype (n8n · Open-Source · Email + Chat)</h1>
    <p class="lead">
      Automatically turn campaign dashboard data into <strong>actionable insights</strong>—delivered as an
      email report and available via a <strong>chat assistant</strong> for follow-up Q&amp;A.
    </p>

    <div class="grid">
      <div class="card">
        <div class="kicker">Live entry point</div>
        <p><a href="https://n8n.rationaleyes.ai/form/pfizer_usecase">n8n Form — Start the flow</a></p>
      </div>
      <div class="card">
        <div class="kicker">Public repo</div>
        <p><a href="https://github.com/kaljuste/campaign_evaluation">github.com/kaljuste/campaign_evaluation</a></p>
      </div>
    </div>

    <div class="hr"></div>

    <h2>What’s inside</h2>
    <ul>
      <li><strong>End-to-end workflow</strong> (n8n), self-hosted with Postgres + Redis + NocoDB; OpenRouter for model access.</li>
      <li><strong>Two AI paths</strong>:
        <ul>
          <li><em>Report Agent</em> → email-ready HTML report with standardized sections and “Actionable Insights”.</li>
          <li><em>Chat Agent</em> → scoped Q&amp;A grounded in the same prepared context pack.</li>
        </ul>
      </li>
      <li><strong>Mock dataset</strong> to simulate monthly performance across markets, channels, specialties, and messages.</li>
    </ul>

    <p class="muted">Screenshot of the n8n flow:</p>
    <p><img src="n8n_workflows.png" alt="n8n workflow screenshot" style="max-width:100%;border:1px solid #e9ecef;border-radius:8px"></p>

    <div class="hr"></div>

    <h2>Quick links</h2>
    <ul>
      <li><a href="https://n8n.rationaleyes.ai/form/pfizer_usecase">Start the flow (form)</a></li>
      <li><a href="https://nocodb.rationaleyes.ai/dashboard/#/nc/view/fe352a7a-160b-4580-9e24-b555b8a2da40">Mock data browser (NocoDB)</a></li>
      <li><strong>Repo files:</strong>
        <ul>
          <li>Workflow: <a href="Campaign_Reporting%20(n8n_workflow).json">Campaign_Reporting (n8n_workflow).json</a></li>
          <li>Mock data CSV: <a href="campaign_data.csv">campaign_data.csv</a></li>
          <li>Prompts:
            <ul>
              <li><a href="report_agent_prompt.md">report_agent_prompt.md</a></li>
              <li><a href="context_agent_prompt.md">context_agent_prompt.md</a></li>
              <li><a href="chat_agent_prompt.md">chat_agent_prompt.md</a></li>
            </ul>
          </li>
          <li>Code (annotated):
            <ul>
              <li><a href="report_aggregation_code%20(JS).md">report_aggregation_code (JS).md</a></li>
              <li><a href="context_normalization_code%20(JS).md">context_normalization_code (JS).md</a></li>
              <li><a href="output_normalization_code%20(JS).md">output_normalization_code (JS).md</a></li>
            </ul>
          </li>
          <li>Screenshot: <a href="n8n_workflows.png">n8n_workflows.png</a></li>
        </ul>
      </li>
    </ul>

    <div class="hr"></div>

    <h2>Business problem &amp; outcome</h2>
    <p><strong>Challenge.</strong> Marketers spend time pulling numbers from multiple dashboards (MOP/Adobe/C360), and still lack crisp, actionable recommendations.</p>
    <p><strong>Pretotype outcome.</strong></p>
    <ul>
      <li><strong>Automates</strong> collect → analyze → <strong>deliver</strong> (email).</li>
      <li>Supports <strong>two-way Q&amp;A</strong> with a chat assistant grounded in the same dataset.</li>
      <li>Produces <strong>standardized</strong>, executive-friendly output answering: What happened? So what? Now what?</li>
    </ul>

    <div class="hr"></div>

    <h2>Scope &amp; assumptions (for speed)</h2>
    <p>
      <span class="pill">Brand: Vydura</span>
      <span class="pill">Campaign: VYDURA_1</span>
      <span class="pill">Markets: Switzerland, Austria</span>
      <span class="pill">Periods: Jun–Aug 2025</span>
      <span class="pill">HCPs: Cardiologists, Neurologists</span>
      <span class="pill">Channels: RTE, Newsletter</span>
      <span class="pill">RTE messages: M1_Efficacy, M2_Safety, M3_Access</span>
    </p>
    <p>The design is <strong>modular and extendable</strong> (add brands, markets, periods, channels, creatives without re-architecting).</p>

    <div class="hr"></div>

    <h2>How it works</h2>
    <ol>
      <li><strong>Form intake</strong> → user selects month &amp; country.</li>
      <li><strong>Aggregation code</strong> summarizes volume &amp; rates; best/worst time slots; specialty/channel splits; message performance.</li>
      <li><strong>Prep/Context agent</strong> converts raw metrics into a compact <em>context pack</em> JSON.</li>
      <li><strong>Report agent</strong> renders a full <em>HTML email</em> with standardized sections and Actionable Insights.</li>
      <li><strong>Email</strong> is sent to the marketer.</li>
      <li><strong>Chat agent</strong> answers follow-up questions using the same context.</li>
    </ol>
    <p class="callout"><strong>Models:</strong> via OpenRouter. In tests, <em>Gemini 2.5 Pro</em> produced strong results (~30s). The chat path can use the same or a lighter model when speed is critical.</p>

    <div class="hr"></div>

    <h2>Why this solves the business challenge</h2>
    <ul>
      <li><strong>Automation:</strong> one request → one email + Q&amp;A.</li>
      <li><strong>Actionable:</strong> consulting appendix/heuristics produce decisions, not just numbers.</li>
      <li><strong>Personalized:</strong> filters narrow to the marketer’s scope.</li>
      <li><strong>Scale-ready:</strong> JSON context pack keeps both paths fast and deterministic.</li>
    </ul>

    <div class="hr"></div>

    <h2>Using the pretotype</h2>
    <h3>Option A — Hosted demo</h3>
    <ol>
      <li>Open the <a href="https://n8n.rationaleyes.ai/form/pfizer_usecase">form</a>.</li>
      <li>Select <em>Period</em> and <em>Country</em>, submit.</li>
      <li>Receive the <strong>HTML report</strong> by email.</li>
      <li>Use chat for questions like:
        <ul>
          <li>“Show main KPIs”</li>
          <li>“Best send times?”</li>
          <li>“How did <em>M2_Safety</em> perform among <em>neurologists</em>?”</li>
          <li>“Compare <em>RTE vs Newsletter</em>”</li>
        </ul>
      </li>
    </ol>

    <h3>Option B — Run locally</h3>
    <ul>
      <li>Import workflow: <a href="Campaign_Reporting%20(n8n_workflow).json">Campaign_Reporting (n8n_workflow).json</a></li>
      <li>Provide connections: Postgres, Redis, NocoDB (optional), OpenRouter API key.</li>
      <li>Select a model (Gemini 2.5 Pro/Flash, GPT-4.1, etc.).</li>
      <li>Trigger via n8n UI or the included Form node.</li>
    </ul>

    <div class="hr"></div>

    <h2>What the report includes</h2>
    <ul>
      <li><strong>Header</strong> (Marketer, Campaign, Period, Date)</li>
      <li><strong>Overview KPIs</strong> (Delivered, Delivery rate, Open rate, Click rate, CTO, Unsub rate, Clicks/1k)</li>
      <li><strong>Send Timing — Best &amp; Worst Slots</strong> (ranked by click rate; low-N filtering)</li>
      <li><strong>RTE Messages</strong> (performance &amp; winner; Δ click-rate in pp)</li>
      <li><strong>Specialty &amp; Channel</strong> comparisons</li>
      <li><strong>Mix of sends</strong> (by channel, specialty, message mix)</li>
      <li><strong>Weekly trend</strong> (delivered, open %, click %, CTO %, unsub %)</li>
    </ul>

    <div class="hr"></div>

    <h2>Design notes &amp; opportunities (Art of the Possible)</h2>
    <ul>
      <li>Connectors for MOP/Adobe/C360; multi-brand/region roll-ups</li>
      <li>A/B testing helper; propose next-month allocations</li>
      <li>Guardrails when unsub/fatigue flags appear</li>
      <li>Goal tracking with traffic-light status</li>
      <li>Comparators vs previous month/market baselines</li>
      <li>Scheduling: monthly/weekly digests; ad-hoc refresh</li>
    </ul>

    <div class="hr"></div>

    <h2>Compliance &amp; privacy</h2>
    <ul>
      <li>Aggregated, anonymized data only; no HCP-identifiable info.</li>
      <li>Prompts avoid medical advice/off-label content.</li>
      <li>Self-hosted, containerized stack for controlled access.</li>
    </ul>

    <div class="hr"></div>

    <h2>File map</h2>
    <ul>
      <li><strong>Workflow</strong>
        <ul><li><a href="Campaign_Reporting%20(n8n_workflow).json">Campaign_Reporting (n8n_workflow).json</a></li></ul>
      </li>
      <li><strong>Prompts</strong>
        <ul>
          <li><a href="report_agent_prompt.md">report_agent_prompt.md</a></li>
          <li><a href="context_agent_prompt.md">context_agent_prompt.md</a></li>
          <li><a href="chat_agent_prompt.md">chat_agent_prompt.md</a></li>
        </ul>
      </li>
      <li><strong>Code (annotated JS)</strong>
        <ul>
          <li><a href="report_aggregation_code%20(JS).md">report_aggregation_code (JS).md</a></li>
          <li><a href="context_normalization_code%20(JS).md">context_normalization_code (JS).md</a></li>
          <li><a href="output_normalization_code%20(JS).md">output_normalization_code (JS).md</a></li>
        </ul>
      </li>
      <li><strong>Data &amp; assets</strong>
        <ul>
          <li><a href="campaign_data.csv">campaign_data.csv</a></li>
          <li><a href="n8n_workflows.png">n8n_workflows.png</a></li>
        </ul>
      </li>
    </ul>

    <div class="hr"></div>

    <h2>FAQ</h2>
    <p><strong>What is RTE?</strong> Rep-Triggered Email — emails initiated by field reps with pre-approved content, sent via the CRM/omnichannel stack.</p>
    <p><strong>Why a “prep/context agent”?</strong> It compacts raw metrics into a stable JSON schema so both the report and chat paths remain fast and deterministic.</p>
    <p><strong>Can this scale?</strong> Yes—schema-driven; add dimensions by updating the aggregation step and forms.</p>

    <p class="footer">© 2025 — Pretotype for interview demonstration. For issues or a walkthrough, reply to the generated email or open a GitHub issue.</p>
  </div>
</body>
</html>
