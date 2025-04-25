import ts from 'typescript';

function extractTextFromTSX(filePath: string): string[] {
  const sourceCode = fs.readFileSync(filePath, 'utf-8');
  const sourceFile = ts.createSourceFile(filePath, sourceCode, ts.ScriptTarget.Latest, true, ts.ScriptKind.TSX);

  const results: string[] = [];

  function visit(node: ts.Node) {
    // 1. 提取 JSX 内部文本，如 <div>按钮</div>
    if (ts.isJsxText(node)) {
      const text = node.getText().trim();
      if (text) results.push(text);
    }

    // 2. 提取字符串字面量，如 title="你好"
    if (ts.isStringLiteral(node)) {
      results.push(node.text.trim());
    }

    ts.forEachChild(node, visit);
  }

  visit(sourceFile);
  return results;
}
