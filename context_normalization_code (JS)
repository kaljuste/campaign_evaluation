// Input: { context_pack: "<potentially truncated JSON string>" }
// Output: { context_pack: <parsed object> }

let s = $json.context_pack;

if (s == null) {
  throw new Error("Missing `context_pack` in input item.");
}

// If already an object, pass through
if (typeof s === 'object') {
  return [{ json: { context_pack: s } }];
}

if (typeof s !== 'string') {
  throw new Error("`context_pack` must be a JSON string or object.");
}

// Clean up the string
s = s.trim();
if (s.startsWith('```')) {
  s = s.replace(/^```[a-zA-Z-]*\n?/, '').replace(/```$/, '').trim();
}

function findLastCompleteStructure(str) {
  console.log(`Analyzing string of length: ${str.length}`);
  console.log(`Last 200 chars: "${str.slice(-200)}"`);
  
  // Find all complete field endings (after closing quotes, after numbers, after booleans)
  let safePositions = [];
  let inString = false;
  let escaped = false;
  let braceDepth = 0;
  let bracketDepth = 0;
  
  for (let i = 0; i < str.length; i++) {
    const char = str[i];
    
    if (escaped) {
      escaped = false;
      continue;
    }
    
    if (char === '\\' && inString) {
      escaped = true;
      continue;
    }
    
    if (char === '"') {
      inString = !inString;
      if (!inString) {
        // Just closed a string - this could be a safe position
        safePositions.push(i + 1);
      }
      continue;
    }
    
    if (inString) continue;
    
    switch (char) {
      case '{':
        braceDepth++;
        break;
      case '}':
        braceDepth--;
        safePositions.push(i + 1);
        break;
      case '[':
        bracketDepth++;
        break;
      case ']':
        bracketDepth--;
        safePositions.push(i + 1);
        break;
      case ',':
        if (braceDepth > 0 || bracketDepth > 0) {
          safePositions.push(i);
        }
        break;
    }
  }
  
  console.log(`Found ${safePositions.length} potential safe positions`);
  
  // Try safe positions from the end backwards
  for (let i = safePositions.length - 1; i >= 0; i--) {
    let pos = safePositions[i];
    let candidate = str.substring(0, pos);
    
    // Skip if too short to be meaningful
    if (candidate.length < 100) continue;
    
    // Count open/close structures in candidate
    let openBraces = (candidate.match(/\{/g) || []).length;
    let closeBraces = (candidate.match(/\}/g) || []).length;
    let openBrackets = (candidate.match(/\[/g) || []).length;
    let closeBrackets = (candidate.match(/\]/g) || []).length;
    
    // Add missing closing braces/brackets
    let testStr = candidate;
    testStr += '}'.repeat(Math.max(0, openBraces - closeBraces));
    testStr += ']'.repeat(Math.max(0, openBrackets - closeBrackets));
    
    console.log(`Trying position ${pos} (${candidate.length} chars)`);
    
    try {
      let parsed = JSON.parse(testStr);
      if (typeof parsed === 'object' && parsed !== null) {
        console.log(`Success at position ${pos}!`);
        return parsed;
      }
    } catch (e) {
      // Continue to next position
    }
  }
  
  // If all else fails, try to find just the header and overall sections
  let headerMatch = str.match(/\{"header":\{[^}]*\}.*?"overall":\{[^}]*\}/);
  if (headerMatch) {
    try {
      let minimal = headerMatch[0] + '}';
      let parsed = JSON.parse(minimal);
      console.log("Recovered minimal structure with header and overall");
      return parsed;
    } catch (e) {
      // Even minimal recovery failed
    }
  }
  
  throw new Error("Could not find any valid JSON structure");
}

// Attempt to parse
let pack;
try {
  pack = JSON.parse(s);
  console.log("JSON parsed successfully on first attempt");
} catch (err) {
  console.log(`Initial parse failed: ${err.message}`);
  
  // Try aggressive truncation
  try {
    pack = findLastCompleteStructure(s);
    console.log("Successfully recovered partial JSON structure");
  } catch (recoveryErr) {
    // Last resort: return a minimal fallback structure
    console.log("All recovery attempts failed, creating fallback structure");
    pack = {
      header: {
        report_name: "Recovered Report",
        error: "Original data was truncated"
      },
      overall: {
        sends: 0,
        delivered: 0,
        error: "Data recovery failed - partial information may be missing"
      },
      recovery_info: {
        original_length: s.length,
        error: err.message,
        recovery_error: recoveryErr.message
      }
    };
  }
}

// Always return something useful
if (typeof pack !== 'object' || pack === null) {
  pack = { error: "Invalid data structure", raw_length: s.length };
}

console.log("Final structure keys:", Object.keys(pack));

return [{ json: { context_pack: pack } }];
