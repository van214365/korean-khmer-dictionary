git commit --allow-empty -m "Trigger rebuild"
git push
<!DOCTYPE html>
<html lang="km">
<head>
  <meta charset="UTF-8" />
  <title>Note App with Editable Flipbook</title>
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <link href="https://fonts.googleapis.com/css2?family=Nokora&family=Moul&family=Hanuman&family=Khmer&family=Preahvihear&family=Battambang&family=Sunflower&family=Nanum+Gothic&display=swap" rel="stylesheet" />
  <style>
    body {
      font-family: 'Hanuman', serif;
      padding: 20px;
      background: #f9f7f1;
      max-width: 700px;
      margin: auto;
    }
    h1 {
      text-align: center;
      color: #5d4037;
    }
    #controls {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-bottom: 10px;
      justify-content: center;
      align-items: center;
    }
    select, input[type="color"] {
      font-size: 16px;
      padding: 5px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    textarea#editor {
      width: 100%;
      height: 100px;
      font-size: 18px;
      padding: 10px;
      border-radius: 5px;
      border: 1px solid #aaa;
      margin-bottom: 10px;
      resize: vertical;
    }
    button {
      padding: 10px 20px;
      background-color: #6d4c41;
      color: white;
      border: none;
      border-radius: 5px;
      cursor: pointer;
      font-weight: bold;
      transition: background-color 0.3s;
    }
    button:hover {
      background-color: #5d3a2e;
    }
    #book {
      margin-top: 20px;
    }
    .page {
      position: relative;
      background: white;
      padding: 20px;
      margin-bottom: 15px;
      border-radius: 8px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.15);
      min-height: 100px;
      word-wrap: break-word;
    }
    .page .controls {
      position: absolute;
      top: 10px;
      right: 10px;
    }
    .page .controls button {
      margin-left: 5px;
      background-color: #8d6e63;
      font-size: 14px;
      padding: 4px 8px;
    }
    .page .controls button:hover {
      background-color: #5d4037;
    }
    .page textarea {
      width: 100%;
      height: 80px;
      font-size: 18px;
      padding: 8px;
      border-radius: 5px;
      border: 1px solid #aaa;
      resize: vertical;
    }
  </style>
</head>
<body>

<h1>✍️ កំណត់អត្ថបទដូចសៀវភៅ (Note + Flipbook)</h1>

<div id="controls">
  <select id="fontSelector" onchange="updateEditorStyle()">
    <option value="Hanuman">Hanuman (ខ្មែរ)</option>
    <option value="Nokora">Nokora (ខ្មែរ)</option>
    <option value="Moul">Moul (ខ្មែរ)</option>
    <option value="Khmer">Khmer (ខ្មែរ)</option>
    <option value="Preahvihear">Preahvihear (ខ្មែរ)</option>
    <option value="Battambang">Battambang (ខ្មែរ)</option>
    <option value="'Sunflower', sans-serif">Sunflower (កូរ៉េ)</option>
    <option value="'Nanum Gothic', sans-serif">Nanum Gothic (កូរ៉េ)</option>
  </select>

  <input type="color" id="colorPicker" onchange="updateEditorStyle()" value="#000000" title="ប្តូរពណ៌អក្សរ" />
  
  <button onclick="addPage()">➕ បន្ថែមទំព័រ</button>
  <button onclick="clearEditor()">🧹 លុបអត្ថបទ</button>
</div>

<textarea id="editor" placeholder="សរសេរអត្ថបទរបស់អ្នក..."></textarea>

<div id="book"></div>

<script>
  const editor = document.getElementById('editor');
  const fontSelector = document.getElementById('fontSelector');
  const colorPicker = document.getElementById('colorPicker');
  const book = document.getElementById('book');

  // Update editor style (font + color)
  function updateEditorStyle() {
    editor.style.fontFamily = fontSelector.value;
    editor.style.color = colorPicker.value;
  }

  // Clear editor content
  function clearEditor() {
    editor.value = '';
  }

  // Create new page in the flipbook
  function addPage() {
    const text = editor.value.trim();
    if (!text) {
      alert('សូមបញ្ចូលអត្ថបទមុន!');
      return;
    }
    createPage(text, fontSelector.value, colorPicker.value);
    editor.value = '';
    savePages();
  }

  // Create page element with controls
  function createPage(text, font, color) {
    const page = document.createElement('div');
    page.className = 'page';
    page.style.fontFamily = font;
    page.style.color = color;

    const contentDiv = document.createElement('div');
    contentDiv.className = 'content';
    contentDiv.textContent = text;

    // Controls: Edit & Delete
    const controls = document.createElement('div');
    controls.className = 'controls';

    const editBtn = document.createElement('button');
    editBtn.textContent = 'Edit';
    editBtn.onclick = () => editPage(page);

    const deleteBtn = document.createElement('button');
    deleteBtn.textContent = 'Delete';
    deleteBtn.onclick = () => {
      if (confirm('តើអ្នកប្រាកដថាចង់លុបទំព័រនេះ?')) {
        page.remove();
        savePages();
      }
    };

    controls.appendChild(editBtn);
    controls.appendChild(deleteBtn);

    page.appendChild(controls);
    page.appendChild(contentDiv);
    book.appendChild(page);
  }

  // Edit page content inline
  function editPage(page) {
    const contentDiv = page.querySelector('.content');
    const oldText = contentDiv.textContent;

    const textarea = document.createElement('textarea');
    textarea.value = oldText;
    textarea.style.fontFamily = page.style.fontFamily;
    textarea.style.color = page.style.color;

    const saveBtn = document.createElement('button');
    saveBtn.textContent = 'Save';
    saveBtn.onclick = () => {
      contentDiv.textContent = textarea.value;
      page.replaceChild(contentDiv, textarea);
      saveBtn.remove();
      savePages();
    };

    page.replaceChild(textarea, contentDiv);
    page.appendChild(saveBtn);
  }

  // Save all pages to localStorage
  function savePages() {
    const pages = [];
    book.querySelectorAll('.page').forEach(page => {
      pages.push({
        text: page.querySelector('.content').textContent,
        font: page.style.fontFamily,
        color: page.style.color
      });
    });
    localStorage.setItem('flipbookPages', JSON.stringify(pages));
  }

  // Load pages from localStorage
  function loadPages() {
    const pages = JSON.parse(localStorage.getItem('flipbookPages')) || [];
    pages.forEach(p => createPage(p.text, p.font, p.color));
  }

  // Initialize
  window.onload = () => {
    updateEditorStyle();
    loadPages();
  };
</script>

</body>
</html>
