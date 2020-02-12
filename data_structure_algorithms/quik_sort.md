# quik_sort

```flow
st=>start: start
e=>end: end
cond_if_bigger_than_base=>condition: if bigger than base data?
op_move_to_next=>operation: move to next in the list
op_set_base=>operation: set the first data is the base
[split to two part(left and right) right elements are bigger than left]
op_insert_left=>operation: insert_left_list(next)
op_insert_right=>operation: insert_right_list(prev)

st->op_set_base->cond_if_bigger_than_base(yes)->op_insert_right->op_move_to_next->cond_if_bigger_than_base
cond_if_bigger_than_base(no)->op_insert_left->op_move_to_next(left)->cond_if_bigger_than_base
```

