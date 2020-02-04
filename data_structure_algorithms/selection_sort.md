# selection_sort

- 第一次认为第一个是最小的和剩余的node比较，如果有更小的就和第一个交换。
- 第二次认为第二个是第二小的和剩余的node比较，如果有更小的就和第二个交换。
- 以此类推，比较完所有的node，完成排序。

```flow
st=>start: start
e=>end: end
cond_sort_all=>condition: sort all?
find_next_min=>operation: find the min in the rest
st->cond_sort_all(no)->find_next_min->cond_sort_all
cond_sort_all(yes)->e
```
## find the min in the rest
```flow
st=>start: start
e=>end: end
cond_find_min=>condition: if the min in the begin?
op_change_pos=>operation: change the position
op_move_next=>operation: move to the next
st->cond_find_min(no)->op_change_pos->op_move_next
cond_find_min(yes)->op_move_next(left)->cond_find_min
```

