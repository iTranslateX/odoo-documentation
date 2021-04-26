# Odoo代码性能测试

本文来自[Odoo 13官方文档之开发者文档](../README.md)系列文章

### ⚠️警告

本教程要求 [已安装Odoo](https://alanhou.org/odoo-13-installing-odoo/) 并 [编写Odoo代码](https://alanhou.org/odoo-13-building-module/)

## 方法的图形化

Odoo内嵌有性能测试器。这一内嵌性能测试器的输出可用于生成由方法所触发调用、查询数、方法内部花费时间及方法及其调用的子方法所花费时间的图表。

```
from odoo.tools.misc import profile
[...]
@profile('/temp/prof.profile')
def mymethod(...)
```

这会生成一个名为/temp/prof.profile的文件

一个名为 *gprof2dot* 的工具会通过这一结果生成图表：

```
gprof2dot -f pstats -o /temp/prof.xdot /temp/prof.profile
```

一个名为 *xdot* 的工具将显示所生成图表：

```
xdot /temp/prof.xdot
```

## 方法日志

另一个性能优化器可用于记录方法的统计数据：

```
from odoo.tools.profiler import profile
[...]
@profile
@api.model
def mymethod(...):
```

一旦待分析的方法完成重审后数据会显示为数据统计日志。

```
2018-03-28 06:18:23,196 22878 INFO openerp odoo.tools.profiler:
calls     queries   ms
project.task ------------------------ /home/odoo/src/odoo/addons/project/models/project.py, 638

1         0         0.02          @profile
                                  @api.model
                                  def create(self, vals):
                                      # context: no_log, because subtype already handle this
1         0         0.01              context = dict(self.env.context, mail_create_nolog=True)

                                      # for default stage
1         0         0.01              if vals.get('project_id') and not context.get('default_project_id'):
                                          context['default_project_id'] = vals.get('project_id')
                                      # user_id change: update date_assign
1         0         0.01              if vals.get('user_id'):
                                          vals['date_assign'] = fields.Datetime.now()
                                      # Stage change: Update date_end if folded stage
1         0         0.0               if vals.get('stage_id'):
                                          vals.update(self.update_date_end(vals['stage_id']))
1         108       631.8             task = super(Task, self.with_context(context)).create(vals)
1         0         0.01              return task

Total:
1         108       631.85
```

## Dump栈

向Odoo进程发送SIGQUIT信号（仅对POSIX可用）会让这个进程输出带有消息级别的当前栈跟踪进行日志。在 odoo进程看起来卡住时，向进程发送这个信号准许来知道进程在做什么，并让进程继续其任务。

## 追踪代码执行

向Odoo进程发送SIGQUIT信号通常足够了，但要查看进程在哪些地方比预期的性能要差的话，我们可以使用 pyflame 工具来替我们执行。

### 安装pyflame和flamegraph

```
# These instructions are given for Debian/Ubuntu distributions
sudo apt install autoconf automake autotools-dev g++ pkg-config python-dev python3-dev libtool make
git clone https://github.com/uber/pyflame.git
git clone https://github.com/brendangregg/FlameGraph.git
cd pyflame
./autogen.sh
./configure
make
sudo make install
```

### 记录已执行代码

既然已经安装了pyflame，现在我们可以使用pyflame来记录已执行代码。这个工具会每秒多次记录进程的 stacktrace（栈追踪）。一旦完成，我们将通过执行图表来进行展示。

```
pyflame --exclude-idle -s 3600 -r 0.2 -p <PID> -o test.flame
```

其中的 <PID>是想要图表化的odoo的进程ID。这会等待直至进程灭亡，最大值为1小时，并每秒获取5次追踪。通过pyflame的输出，我们可以使用flamegraph工具生成一张SVG 图表：

```
flamegraph.pl ./test.flame > ~/mycode.svg
```

![img](https://www.odoo.com/documentation/13.0/_images/flamegraph.svg)