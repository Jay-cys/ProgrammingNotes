
# 函数使用格式
makefile中的函数时事先定义好的，被make命令所支持，使用的格式如下：

- `$(函数名 参数,参数..)`
- `${函数名 参数,参数..}`



# 常用函数

- subst：负责字符串替换功能，将text中from内容替换为to，格式：`$(subst <from>,<to>,<text>)`
- patsubst：通配符字符串替换，格式：`$(patsubst <pattern>,<replacement>,<text>)`
- dir：从文件路径中获取目录，格式：`$(dir <names……>)`
- notdir：从文件路径中获取文件名，格式：`$(notdir <names……>)`
- wildcard：展开通配符代表的含义，可以用在模式规则之外，用在变量定义和函数中，格式：`$(wildcard PATTERN……)`


