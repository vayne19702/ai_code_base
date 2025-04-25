import ts from 'typescript';

function extractTextFromTSX(filePath: string): string[] {
  const sourceCode = fs.readFileSync(filePath, 'utf-8');
  const sourceFile = ts.createSourceFile(filePath, sourceCode, ts.ScriptTarget.Latest, true, ts.ScriptKind.TSX);

  const results: string[] = [];

  function visit(node: ts.Node, parent: ts.Node | undefined) {
    // 提取 JSX 标签中的纯文本 <div>欢迎</div>
    if (ts.isJsxText(node)) {
      const text = node.getText().trim();
      if (text) {
        results.push(text);
      }
    }

    // 提取字符串字面量（如 title="你好"）
    if (ts.isStringLiteral(node)) {
      // 过滤 import "xx" 或 import xx from "xx"
      if (parent && ts.isImportDeclaration(parent)) return;

      // 过滤 className、id、style 这类常见的非翻译属性
      if (
        parent &&
        ts.isJsxAttribute(parent) &&
        ['className', 'id', 'style', 'src', 'href'].includes(parent.name.text)
      ) {
        return;
      }

      const text = node.text.trim();
      if (text) {
        results.push(text);
      }
    }

    ts.forEachChild(node, (child) => visit(child, node));
  }

  visit(sourceFile, undefined);
  return results;
}
