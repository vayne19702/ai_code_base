if (
  parent &&
  ts.isJsxAttribute(parent) &&
  parent.name &&
  ts.isIdentifier(parent.name)
) {
  const attrName = parent.name.escapedText?.toString();
  const ignoredAttrs = ['className', 'id', 'style', 'src', 'href'];

  if (attrName && ignoredAttrs.includes(attrName)) {
    return; // 跳过这些属性值
  }
}
