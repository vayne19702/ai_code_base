plugins.push(
  require('@babel/plugin-transform-react-jsx'),
  require('@babel/plugin-transform-react-display-name'),
  require('@babel/plugin-transform-typescript'),

  // 支持 class 属性（如 class fields）
  require('@babel/plugin-proposal-class-properties'),

  // 支持私有字段（#xxx）
  require('@babel/plugin-proposal-private-methods'),

  // 支持可选链（a?.b）
  require('@babel/plugin-proposal-optional-chaining'),

  // 支持空值合并运算符（a ?? b）
  require('@babel/plugin-proposal-nullish-coalescing-operator'),

  // 支持装饰器（如 @observer）
  [require('@babel/plugin-proposal-decorators'), { legacy: true }]
);



npm install --save-dev \
  @babel/plugin-transform-react-jsx \
  @babel/plugin-transform-react-display-name \
  @babel/plugin-transform-typescript \
  @babel/plugin-proposal-class-properties \
  @babel/plugin-proposal-private-methods \
  @babel/plugin-proposal-optional-chaining \
  @babel/plugin-proposal-nullish-coalescing-operator \
  @babel/plugin-proposal-decorators


