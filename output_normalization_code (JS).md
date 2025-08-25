// Get the model output (adjust if your field name is different)
let raw = $json.output || $json.text || $json;

// Make sure it's a string
raw = String(raw);

// Strip leading/trailing Markdown fences if they exist
raw = raw.replace(/^```[a-zA-Z]*\n?/, "").replace(/```$/, "");

// Return cleaned HTML string
return [{ json: { html: raw } }];
