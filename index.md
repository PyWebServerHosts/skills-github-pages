<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>TagScript Live Preview Editor</title>
  <link href="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/themes/prism.min.css" rel="stylesheet"/>
  <style>
    body { margin: 0; font-family: sans-serif; display: flex; height: 100vh; }
    textarea { width: 50%; padding: 10px; font-family: monospace; font-size: 14px; border: none; outline: none; }
    #output { width: 50%; padding: 10px; overflow: auto; background: #f9f9f9; }
    pre code { padding: 10px; display: block; border-radius: 5px; background: #eee; }
    .note { border-left: 4px solid #2196F3; padding: 10px; background: #f1f1f1; margin: 5px 0; }
    .button { padding: 8px 16px; background: #008CBA; color: white; border: none; cursor: pointer; text-decoration: none; display: inline-block; }
    .highlight { background: yellow; font-weight: bold; padding: 2px 4px; }
    .badge { padding: 2px 6px; border-radius: 12px; color: white; font-size: 0.8em; }
    .tooltip { position: relative; cursor: pointer; border-bottom: 1px dotted black; }
    .tooltip:hover::after {
      content: attr(data-tip);
      position: absolute;
      left: 0; top: 100%;
      background: #333; color: #fff; padding: 4px 8px;
      font-size: 0.8em; white-space: nowrap;
      border-radius: 4px;
      z-index: 10;
    }
    .collapse { cursor: pointer; font-weight: bold; }
    .collapse + div { display: none; padding: 5px; border: 1px solid #ccc; margin-top: 4px; }
  </style>
</head>
<body>

<textarea id="input" placeholder="Write your TagScript here...">
{@heading, "Live Preview", _design = #level="2"}
{@code, --#lang-js&code="console.log('Hello, world!');", "Executecodebutton?"="true", "ClearCodeButton?"="true"}
{@tooltip, "Hover me", _design = #tip="This is a tooltip!"}
{@badge, "Beta", _design = #color="purple"}
{@collapse, "Click to see more", _design = #content="This is hidden!"}
{@divider}
</textarea>

<div id="output"></div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/prism.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-javascript.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-css.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/prism/1.29.0/components/prism-markup.min.js"></script>

<script>
  const input = document.getElementById('input');
  const output = document.getElementById('output');

  input.addEventListener('input', render);
  document.addEventListener('DOMContentLoaded', render);

  function render() {
    output.innerHTML = parseTagScript(input.value);
    Prism.highlightAll();

    document.querySelectorAll('.collapse').forEach(trigger => {
      trigger.addEventListener('click', () => {
        const content = trigger.nextElementSibling;
        content.style.display = content.style.display === 'block' ? 'none' : 'block';
      });
    });

    // Handle code block execution and clearing
    document.querySelectorAll('.exec-code-btn').forEach(button => {
      button.addEventListener('click', function() {
        const code = this.closest('.code-block').querySelector('code').textContent;
        try {
          eval(code); // Execute the code
        } catch (error) {
          console.error('Error executing code:', error);
        }
      });
    });

    document.querySelectorAll('.clear-code-btn').forEach(button => {
      button.addEventListener('click', function() {
        this.closest('.code-block').querySelector('code').textContent = '';
      });
    });
  }

  function parseTagScript(text) {
    // Handle tag-only commands (like divider, etc.)
    text = text.replace(/\{@(divider)\}/g, (_, cmd) => {
      if (cmd === 'divider') return `<hr>`;
      return '';
    });

    // Handle code block with execution and clear buttons
    text = text.replace(/\{@code,\s*--#lang-(\w+)&code="([^"]*)"(?:,\s*"Executecodebutton\?"="(true|false)")?(?:,\s*"ClearCodeButton\?"="(true|false)")?\}/g, 
      (_, lang, code, execBtn, clearBtn) => {
        let langClass = lang === 'html' ? 'language-markup'
                    : (lang === 'css' || lang === 'design-css') ? 'language-css'
                    : 'language-javascript';

        let executeButton = execBtn === 'true' ? `<button class="exec-code-btn">Execute Code</button>` : '';
        let clearButton = clearBtn === 'true' ? `<button class="clear-code-btn">Clear Code</button>` : '';

        return `
          <div class="code-block">
            <pre><code class="${langClass}">${escapeHTML(code)}</code></pre>
            ${executeButton} ${clearButton}
          </div>
        `;
      });

    // Handle other TagScript commands
    return text.replace(/\{@(\w+),\s*"([^"]*)"(?:,\s*_design\s*=\s*#(\w+-?\w*)="([^"]*)")?\}/g,
      (_, cmd, content, key, value) => {
        switch (cmd) {
          case 'bold': return `<strong>${content}</strong>`;
          case 'italic': return `<em>${content}</em>`;
          case 'underline': return `<u>${content}</u>`;
          case 'link': return `<a href="${value}" target="_blank"><strong>${content}</strong></a>`;
          case 'heading': return `<h${value || 2}>${content}</h${value || 2}>`;
          case 'color': return `<span style="color:${value}">${content}</span>`;
          case 'bgcolor': return `<span style="background-color:${value}">${content}</span>`;
          case 'note': return `<div class="note">${content}</div>`;
          case 'image': return `<img src="${value}" alt="${content}" />`;
          case 'button': return `<a href="${value}" target="_blank" class="button">${content}</a>`;
          case 'quote': return `<blockquote>${content}</blockquote>`;
          case 'list': return `<ul>${content.split(',').map(i => `<li>${i.trim()}</li>`).join('')}</ul>`;
          case 'checklist': return `<ul>${content.split(',').map((i, idx) =>
            `<li><input type="checkbox" ${value == idx ? 'checked' : ''}/> ${i.trim()}</li>`).join('')}</ul>`;
          case 'video': return `<video src="${value}" controls>${content}</video>`;
          case 'audio': return `<audio controls src="${value}">${content}</audio>`;
          case 'tooltip': return `<span class="tooltip" data-tip="${value}">${content}</span>`;
          case 'highlight': return `<span class="highlight">${content}</span>`;
          case 'badge': return `<span class="badge" style="background:${value}">${content}</span>`;
          case 'collapse': return `<div class="collapse">${content}</div><div>${value}</div>`;
          default: return content;
      }
    });
  }

  function escapeHTML(str) {
    return str.replace(/[&<>"']/g, tag => ({
      '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;'
    }[tag]));
  }
</script>

</body>
</html>
