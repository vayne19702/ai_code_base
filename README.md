import fetch from 'node-fetch';  // 注意：VSCode扩展里要用 node-fetch（不是浏览器fetch）

export type TranslationItem = {
  content: string;
  translation: {
    language: string;
    value: string;
  }[];
};

/**
 * 调用翻译API，返回带翻译结果的数组
 * @param contents 原文数组
 * @returns 包含翻译结果的数组
 */
export async function fetchTranslations(contents: string[]): Promise<TranslationItem[]> {
  const response = await fetch('https://your-api.com/translate', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      texts: contents,
      targetLanguages: ['chinese', 'japanese']  // 按需修改
    })
  });

  if (!response.ok) {
    throw new Error(`Translation API call failed with status ${response.status}`);
  }

  const result: TranslationItem[] = await response.json();
  return result;
}
