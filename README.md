export function activate(context: vscode.ExtensionContext) {
    const scanProvider = new ScanProvider();
    vscode.window.registerTreeDataProvider('staticTextScanner', scanProvider);

    vscode.commands.registerCommand('staticTextScanner.scan', async () => {
        scanProvider.startScanning();
    });

    vscode.commands.registerCommand('staticTextScanner.openFile', (item: ScanItem) => {
        openResultDocument(item);
    });
}

async function openResultDocument(item: ScanItem) {
    const doc = await vscode.workspace.openTextDocument({ content: item.content, language: 'plaintext' });
    await vscode.window.showTextDocument(doc, { preview: false });
}





import * as vscode from 'vscode';
import * as path from 'path';

export class ScanProvider implements vscode.TreeDataProvider<ScanItem> {
    private _onDidChangeTreeData: vscode.EventEmitter<ScanItem | undefined | void> = new vscode.EventEmitter<ScanItem | undefined | void>();
    readonly onDidChangeTreeData: vscode.Event<ScanItem | undefined | void> = this._onDidChangeTreeData.event;

    private items: ScanItem[] = [];
    private isScanning: boolean = false;

    constructor() {}

    getTreeItem(element: ScanItem): vscode.TreeItem {
        return element;
    }

    getChildren(element?: ScanItem): Thenable<ScanItem[]> {
        if (this.isScanning) {
            return Promise.resolve([
                new ScanItem('Scanning...', '', vscode.TreeItemCollapsibleState.None)
            ]);
        }
        return Promise.resolve(this.items);
    }

    async startScanning() {
        this.isScanning = true;
        this.refresh();

        // 模拟扫描过程
        const files = await vscode.workspace.findFiles('**/*.jsx', '**/node_modules/**');

        const scanResults: ScanItem[] = [];
        for (const file of files) {
            const content = await vscode.workspace.fs.readFile(file);
            const text = content.toString();

            // 简单模拟：取出所有"xxx"中的静态文本
            const matches = Array.from(text.matchAll(/"([^"]+)"/g));

            if (matches.length > 0) {
                const resultText = matches.map(m => m[1]).join('\n');
                const item = new ScanItem(
                    path.relative(vscode.workspace.rootPath || '', file.fsPath),
                    resultText,
                    vscode.TreeItemCollapsibleState.None
                );
                scanResults.push(item);
            }
        }

        this.items = scanResults;
        this.isScanning = false;
        this.refresh();
    }

    refresh(): void {
        this._onDidChangeTreeData.fire();
    }
}

export class ScanItem extends vscode.TreeItem {
    constructor(
        public readonly label: string,
        public readonly content: string,
        public readonly collapsibleState: vscode.TreeItemCollapsibleState
    ) {
        super(label, collapsibleState);
        this.command = {
            command: 'staticTextScanner.openFile',
            title: 'Open File',
            arguments: [this]
        };
    }
}
