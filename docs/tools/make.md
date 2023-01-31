# Make

## 遍历文件

```makefile
dirs := $(shell find . -maxdepth 3 -type d)
files := $(foreach dir,$(dirs),$(wildcard $(dir)/*.c))
objs := $(patsubst %.c,%.o, $(FILES))
```

## 变量的替换引用

```makefile
# $(VAR:A=B)
# example
# 效果和patsust是一样的
foo := a.o b.o c.o
bar := $(foo:%.o=%.c)
```

## 将中间文件输出到指定的目录

将中间文件输出到指定的目录需要将构建规则一并进行修改，如下示例可以将中间文件输出到`OUTPUT`指定的目录中去

```makefile
OUTPUT := build

# 需要改写这里的匹配规则并增加下面创建对应目录的操作
$(OUTPUT)/%.o: %.c
	@echo "  CC  $<"
	@if [ ! -d $(dir $@) ]; then mkdir -p $(dir $@); fi
	@$(CC) -c -o $@ $< $(CFLAGS)
```

## 头文件改动后不会重新编译的解决办法

gcc提供了一个叫做`-M`的选项用来生成make格式的源码依赖文件，我们可以在编译选项中增加`-MMD`,`-MP`两个参数来自动生成和源文件名称相同并且后缀为`.d`的依赖文件并在顶层Makefile中使用`-include $(deps)`来包含这些规则，这样当`.h`文件修改后便会自动编译需要重新编译的文件了，使用示例如下:

```makefile
CFLAGS += -MMD -MP

objs := $(patsubst %.c,$(OUTPUT)/%.o,$(srcs))
deps := $(objs:%.o=%.d)

-include $(deps)
$(OUTPUT)/%.o: %.c
	@echo "  CC  $<"
	@if [ ! -d $(dir $@) ]; then mkdir -p $(dir $@); fi
	@$(CC) -c -o $@ $< $(CFLAGS)
```

## 简单的模板

```Makefile
# Makefile
CROSS :=

CC = $(CROSS)gcc
STRIP = $(CROSS)strip

objs := a.o
objs += lib/b.o

c-flags := -ffunction-sections -fdata-sections
ld-flags := -Wl,-gc-sections

target: $(objs)
    @CC $(ld-flags) -o $@ $^
    @STRIP $@

$(objs): %.o: %.c
    @CC $(c-flags) -c -o $@ $^

.PHONY: clean

clean:
    @find -name '*.o' | xargs rm -f
```
