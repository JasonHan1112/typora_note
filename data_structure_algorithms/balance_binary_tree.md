# balance_binary_tree
- 递归创建二叉树
```flow
st=>start: start
e=>end: end
op_set_data=>operation: set data with keyboard(input 0 to return NULL)
op_create_left_tree=>operation: create left sub tree(create_binary_node)
op_create_right_tree=>operation: create right sub tree(create_binary_node)
op_recursion=>parallel: ***************(create_binary_node)
pa=>parallel:
cond_l=>condition: with left sub tree？
cond_r=>condition: with right sub tree？

st->op_set_data->op_create_left_tree->op_create_right_tree->op_recursion(path1, right)->e
op_recursion(path2, left)->op_set_data
```
- 递归计算节点数
```flow
st=>start: start
e=>end: end
op_find_leaf_node=>operation: find leaf node return 0
op_sum_node=>parallel: return sum(left)+sum(right)(recursion)+1
st->op_find_leaf_node->op_sum_node(path1)->e
op_sum_node(path2, right)->op_find_leaf_node
```