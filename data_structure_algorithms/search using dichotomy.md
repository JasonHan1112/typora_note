# search using dichotomy

```flow
st=>start: start
ed1=>end: return data
ed2=>end: return nothing
o_half=>operation: get the half pos
o_change_to_left=>operation: search left half array
o_change_to_right=>operation: search right half
cond_if_eq=>condition: array[half]==the data?
cond_right_or_left=>condition: array[half]>the data?
cond_cannt_find=>condition: search done(header>tail)?

st->cond_cannt_find(no, left)->o_half->cond_if_eq(yes,right)->ed1
cond_if_eq(no)->cond_right_or_left(yes,right)->o_change_to_left(right)->cond_cannt_find
cond_right_or_left(no)->o_change_to_right(left)->cond_cannt_find
cond_cannt_find(yes)->ed2
```

