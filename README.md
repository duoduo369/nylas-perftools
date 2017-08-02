This is a collection of minimalist utilities for profiling Python programs. The motivation behind them is described in our [blog post](https://www.nylas.com/blog/performance).


# py2devtools
The profile visualizer that's built into the Chrome developer tools is pretty rad. `py2devtools.py` contains instrumentation to create a `.cpuprofile` file from a Python program that can be loaded into the developer tools. See the module docstring for details.


# stacksampler

`stacksampler.py` contains a sampling profiler, along with a minimal embedded HTTP server to expose its data. It's built to work with [gevented](https://github.com/gevent/gevent) applications, but can be adapted to work without. Assuming gevent, drop

```
import stacksampler
gevent.spawn(stacksampler.run_profiler)
```

into your code, run your application, and then do

```
curl localhost:16384
```

to get profiling data. See the module docstring for more details.


# The stackcollector agent

![Screenshot](/images/screenshot.png)

The `stackcollector` package adds basic support for automatically collecting and visualizing profiles from distributed processes. It has two parts: a long-running collector agent that periodically gets samples from processes, and a frontend that serves visualizations. Data is timestamped and persisted using gdbm, allowing for time-based querying.

## Installation

```
# create a directory for data files
sudo mkdir -p /var/lib/stackcollector
sudo chmod a+rw /var/lib/stackcollector

virtualenv .
source bin/activate
python setup.py install
```

## Running the collector

The collector assumes that processes expose profiles in the [flamegraph line format](https://github.com/brendangregg/FlameGraph#2-fold-stacks) over HTTP, as implemented by `stacksampler.py`.

```
# Every minute, gather stacks from a local process listening on port 16384.
python -m stackcollector.collector --host localhost --ports 16384 --interval 60
```

## Running the visualizer

```
python -m stackcollector.visualizer --port 5555
```

Then visit e.g. `http://localhost:5555?from=-15minutes` to see data from the past 15 minutes.

# Questions? Issues?

Don't hesitate to get in touch!


# 真正应该这么做

    git clone 这个项目
    cd 到项目目录
    pip install -e .

    将 stacksampler.py 复制到项目目录，在按照 readme
    中代码修改的方式修改对应代码

    项目启动后执行
    python -m stackcollector.collector --host localhost --ports 16384 --interval 60
    写一个批量请求脚本 test.sh
    每秒执行
    watch -n1 test.sh

    可视化工具
    python -m stackcollector.visualizer --port 5555
    如果项目所在机器无法通过5555端口访问，将 /var/lib/stackcollector
    下的所有文件复制到可以访问机器访问
