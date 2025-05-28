function splitObjectByMaxKeysPerBatch<T>(obj: { [key: string]: T }, maxKeysPerBatch: number): { [key: string]: T }[] {
  const keys = Object.keys(obj);
  const result: { [key: string]: T }[] = [];

  for (let i = 0; i < keys.length; i += maxKeysPerBatch) {
    const batch: { [key: string]: T } = {};
    const chunkKeys = keys.slice(i, i + maxKeysPerBatch);

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
