# configfs简介
- what is configfs
1. ram-based file system manager of kernel objects.
2. created via an explicit userspace operation: mkdir, rmdir, symlink. 
3. the lifetime of the representation is completely driven by userspace.
4. both sysfs and configfs can and should exist together on the system.
- using configfs
configfs can be compiled as a module or into the kernel. you can access it by doing:
```c
mount -t configfs none /config
```
once a client subsystem is loaded, it will appear as a subdirectory under /config. an item is create via mkdir. the item's attributes will also appear at this time. readdir can determine what the attributes are, read can query their default values, write can store new values.
- configfs attributes
1. normal attributes:
small ascii text files, with a maximum size of one page. configfs store the entire buffer at once
2. binary attributes:
page_size limitation does not apply, but the whole binary item must fit in single kernel vmalloc buffer. the wirte calls from userspace are buffered.

- coding with configfs
1. configfs_subsystem
a subsystem consists of a toplevel config_group and a mutex. configfs_register_subsystem(),
```c
struct configfs_subsystem {
        struct config_group     su_group;
        struct mutex            su_mutex;
};

int configfs_register_subsystem(struct configfs_subsystem *subsys);
void configfs_unregister_subsystem(struct configfs_subsystem *subsys);
```
2. config_group
config_item live in a config_group, the only way one can be created is via mkdir on a config_group, config_group contains a config_item. a group can create child items or groups. this is accomplished via the group operations specified on the group's config_item_type
```c
struct config_group {
        struct config_item              cg_item;
        struct list_head                cg_children;
        struct configfs_subsystem       *cg_subsys;
        struct list_head                default_groups;
        struct list_head                group_entry;
};

void config_group_init(struct config_group *group);
void config_group_init_type_name(struct config_group *group,
                                 const char *name,
                                 struct config_item_type *type);

struct configfs_group_operations {
        struct config_item *(*make_item)(struct config_group *group,
                                         const char *name);
        struct config_group *(*make_group)(struct config_group *group,
                                           const char *name);
        int (*commit_item)(struct config_item *item);
        void (*disconnect_notify)(struct config_group *group,
                                  struct config_item *item);
        void (*drop_item)(struct config_group *group,
                          struct config_item *item);
};
```
3. config_item
config_item is embedded in a container structure, a structure represents what the subsystem is doing.
```c
struct config_item {
        char                    *ci_name;
        char                    ci_namebuf[UOBJ_NAME_LEN];
        struct kref             ci_kref;
        struct list_head        ci_entry;
        struct config_item      *ci_parent;
        struct config_group     *ci_group;
        struct config_item_type *ci_type;
        struct dentry           *ci_dentry;
};

void config_item_init(struct config_item *);
void config_item_init_type_name(struct config_item *,
                                const char *name,
                                struct config_item_type *type);
struct config_item *config_item_get(struct config_item *);
void config_item_put(struct config_item *);
```
4. config_item_type
```c
struct configfs_item_operations {
        void (*release)(struct config_item *);
        int (*allow_link)(struct config_item *src,
                          struct config_item *target);
        void (*drop_link)(struct config_item *src,
                         struct config_item *target);
};

struct config_item_type {
        struct module                           *ct_owner;
        struct configfs_item_operations         *ct_item_ops;
        struct configfs_group_operations        *ct_group_ops;
        struct configfs_attribute               **ct_attrs;
        struct configfs_bin_attribute           **ct_bin_attrs;
};
```
5. configfs_attribute and configsfs_bin_attribute
```c
struct configfs_attribute {
        char                    *ca_name;
        struct module           *ca_owner;
        umode_t                  ca_mode;
        ssize_t (*show)(struct config_item *, char *);
        ssize_t (*store)(struct config_item *, const char *, size_t);
};

struct configfs_bin_attribute {
        struct configfs_attribute       cb_attr;
        void                            *cb_private;
        size_t                          cb_max_size;
};

```
6. example linux/samples/configs/configfs_sample.c
