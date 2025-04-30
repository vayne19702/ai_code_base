const scanResults = [
  {
    content: "Save",
    translation: [
      { language: "Chinese", value: "保存" },
      { language: "Japanese", value: "保存する" }
    ]
  },
  {
    content: "Cancel",
    translation: [
      { language: "Chinese", value: "取消" },
      { language: "Japanese", value: "キャンセル" }
    ]
  }
];



function generateStaticHtml(scanResults: any[]): string {
  const itemsHtml = scanResults.map(item => {
    const translations = item.translation.map(t => 
      `<div class="translation">${t.language}: ${t.value}</div>`
    ).join("");

    return `
      <div class="item">
        <div class="content">${item.content}</div>
        ${translations}
      </div>
    `;
  }).join("");

  return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <style>
        body { font-family: sans-serif; padding: 1em; }
        .item { margin-bottom: 1.5em; }
        .content { font-weight: bold; margin-bottom: 0.3em; white-space: pre-wrap; }
        .translation { margin-left: 1em; color: #555; }
      </style>
    </head>
    <body>
      <h2>Scan Results</h2>
      ${itemsHtml}
    </body>
    </html>
  `;
}
