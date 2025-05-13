import * as vscode from 'vscode';
import * as path from 'path';

let panel: vscode.WebviewPanel | undefined;

export function showNewLanguagePanel(extensionUri: vscode.Uri) {
  if (panel) {
    panel.reveal(vscode.ViewColumn.One);
    return;
  }

  panel = vscode.window.createWebviewPanel(
    'newLanguage',
    'Add New Language',
    vscode.ViewColumn.One,
    {
      enableScripts: true,
      localResourceRoots: [vscode.Uri.joinPath(extensionUri, 'media')],
    }
  );

  const html = getHtml(panel.webview, extensionUri);
  panel.webview.html = html;

  panel.webview.onDidReceiveMessage(async (message) => {
    if (message.type === 'addLanguage') {
      const targetLang = message.targetLang;
      vscode.window.showInformationMessage(`Adding new language: ${targetLang}`);

      // TODO: 调用翻译 API 和写入 JSON 文件逻辑
    }
  });

  panel.onDidDispose(() => {
    panel = undefined;
  });
}

function getHtml(webview: vscode.Webview, extensionUri: vscode.Uri): string {
  const scriptUri = webview.asWebviewUri(
    vscode.Uri.joinPath(extensionUri, 'media', 'newLanguage.js')
  );

  return /* html */ `
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Add New Language</title>
    </head>
    <body>
      <h2>Add New Language</h2>
      <label>
        Language Code:
        <input type="text" id="langCode" />
      </label>
      <br /><br />
      <button id="submitBtn">🌐 Add</button>

      <script src="${scriptUri}"></script>
    </body>
    </html>
  `;
}
