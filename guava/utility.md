1. Don't use NULL;
  1.1 null 会 造成NPE
  1.2 null 经常会带来歧义, 比如某个map.get(key) => null, 无法直接判断是该key不存在，还是值就是null；
  1.3 可以用Optional代替null