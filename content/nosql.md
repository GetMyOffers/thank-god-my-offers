## 1. Redis有几种数据结构，并且对于hash数据结构，如果要删除数据，那么redis的底层是怎么处理的

1. redis 数据结构

   - string
   - list
   - hash
   - set
   - sort set

2. hash 数据结构,删除数据

   redis提供的 hash, 底层实现就是一个 hash table,通过拉链法解决 hash 冲突.

   ```c
       for (table = 0; table <= 1; table++) {
           idx = h & d->ht[table].sizemask;
           he = d->ht[table].table[idx];
           prevHe = NULL;
           while(he) {
               if (key==he->key || dictCompareKeys(d, key, he->key)) {
                   /* Unlink the element from the list */
                   if (prevHe)
                       prevHe->next = he->next;
                   else
                       d->ht[table].table[idx] = he->next;
                   if (!nofree) {
                       dictFreeKey(d, he);
                       dictFreeVal(d, he);
                       zfree(he);
                   }
                   d->ht[table].used--;
                   return he;
               }
               prevHe = he;
               he = he->next;
           }
           if (!dictIsRehashing(d)) break;
       }
   ```

   