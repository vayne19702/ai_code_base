function splitObjectIntoBatches<T>(obj: { [key: string]: T }, batchCount: number): { [key: string]: T }[] {
  const keys = Object.keys(obj);
  const batchSize = Math.ceil(keys.length / batchCount);

  const result: { [key: string]: T }[] = [];

  for (let i = 0; i < batchCount; i++) {
    const batch: { [key: string]: T } = {};
    const start = i * batchSize;
    const end = start + batchSize;
    const chunkKeys = keys.slice(start, end);

    chunkKeys.forEach(key => {
      batch[key] = obj[key];
    });

    result.push(batch);
  }

  return result;
}


type Result = { [key: string]: number[] };

const result: Result = {
  a: [1, 2],
  b: [3, 4],
  c: [5, 6],
  d: [7, 8],
  e: [9, 10]
};

const batches = splitObjectIntoBatches(result, 2); // 分成 2 个批次
console.log(batches);
