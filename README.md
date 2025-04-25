import ts from 'typescript';

function extractWithTSCompiler(filePath: string): string[] {
  const sourceCode = fs.readFileSync(filePath, 'utf-8');
  const sourceFile = ts.createSourceFile(filePath, sourceCode, ts.ScriptTarget.Latest, true);

  const results: string[] = [];

  function visit(node: ts.Node) {
    if (ts.isStringLiteral(node)) {
      results.push(node.text);
    }
    ts.forEachChild(node, visit);
  }

  visit(sourceFile);
  return results;
}
