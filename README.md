export class ScanItem extends vscode.TreeItem {
    constructor(
        public readonly label: string,
        public readonly filePath: string,
        public readonly scanResult: string,
        public readonly collapsibleState: vscode.TreeItemCollapsibleState
    ) {
        super(label, collapsibleState);
        this.command = {
            command: 'staticTextScanner.openFileWithScanResult',
            title: 'Open File With Scan Result',
            arguments: [this]
        };
    }
}


vscode.commands.registerCommand('staticTextScanner.openFileWithScanResult', async (item: ScanItem) => {
    const doc = await vscode.workspace.openTextDocument(item.filePath);
    await vscode.window.showTextDocument(doc, { preview: false });

    const panel = vscode.window.createWebviewPanel(
        'scanResultView',
        `Scan Result: ${item.label}`,
        vscode.ViewColumn.Beside,
        { enableScripts: true }
    );

    panel.webview.html = getWebviewContent(item.scanResult);
});



function getWebviewContent(content: string): string {
    return `
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>Scan Result</title>
        <style>
            body { font-family: sans-serif; padding: 1rem; }
            pre { background: #f0f0f0; padding: 1rem; border-radius: 8px; }
        </style>
    </head>
    <body>
        <h2>Scan Result</h2>
        <pre>${content}</pre>
    </body>
    </html>
    `;
}
