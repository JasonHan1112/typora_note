# IDR数据结构
```c
struct idr {
    struct radix_tree_root  idr_rt;
    unsigned int        idr_base;
    unsigned int        idr_next;
};  
```
- 绑定的一种数据结构。也称为关联数组。可以视为由唯一key组成的集合。每个key对应这一个value。通过radix tree来映射相应的key value
```c
/*为ptr分配一个id（key value between start and end*/
int idr_alloc(struct idr *idr, void *ptr, int start, int end, gfp_t gfp)
/*根据给入的id，返回相对应的指针（之前添加的）*/
void *idr_find(const struct idr *idr, unsigned long id)
/*从idr中删除一个id*/
void *idr_remove(struct idr *idr, unsigned long id)
/*删除整个idr以及其占用的内存*/
void idr_destroy(struct idr *idr)
```
- radix tree