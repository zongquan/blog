---
layout: post
title:  "Lua Deepcopy引起的内存泄露"
author: "Zongquan"
comments: false
---

这些天，[《攻略三国》](http://3g.gyyx.cn/)开始导入玩家线上测试，晚上接到领导的电话，说游戏出现了反复重启的情况，急忙回到公司，ssh到服务器上，发现游戏的Logic进程占用的内存在随着时间的不断升高，最终多个进程把系统的内存占满后，被Kill掉，GameMgr检测到Logic进程挂了后会拉起进程，就出现了反复重启的情况。

上线前，压测的机器人已经是持续模拟了2000玩家同时在线操作各种业务逻辑的场景，稳定跑了超过一周的时间的，并没有出现该问题，服务器的各项指标都比较稳定。

我们的游戏服务器是一个C++ + lua的架构，2011年公司打算做网页游戏的时候，全全由我来主导开发的游戏服务器引擎。该引擎对于客户端的每一次操作都会转化成一个事件，交给Lua处理，同时在C++里会把每一个事件的处理信息记录的Log中，包括处理这个事件Lua占用的时间，Lua_state的内存增量等信息。

从Log里明显看到有一个事件ID，在每次该事件ID处理后，Lua_state的内存会增加1MB左右，一个非常恐怖的增量数字。通过ID分析，才发现是一个近期添加的新功能的刷新操作，而该功能正好也没有更新到压测的bot工具里，压测没有发现。

分析代码，发现问题的根源是同事使用了deepcopy函数导致。

```lua
--深度拷贝
function deepcopy(object)    
	local lookup_table = {}
	local function _copy(object)
		if type(object) ~= "table" then
			return object
		elseif lookup_table[object] then
			return lookup_table[object]
		end  -- if        
		local new_table = {}
		lookup_table[object] = new_table
		for index, value in pairs(object) do
			new_table[_copy(index)] = _copy(value)
		end  -- for        
		return setmetatable(new_table, getmetatable(object))    
	end  -- function _copy    
	return _copy(object)
end  -- function deepcopy
```

每次刷新的时候，调用该函数对一个对象进行深度拷贝。

对于一个普通的lua table来说，如果不存在**交叉引用**是不会有问题的。但当一个table里存在**交叉引用**的时候，用deepcopy就会出现严重的内存泄露，实际证明，deepcopy并不会因为有交叉引用的存在进入一个死循环导致栈溢出的异常，反而成功返回了一个拷贝对象，但拷贝的数据量非常大。打印出来看，这是一个非常deep的一个table，大概就是1MB左右。

改成其它方式后，问题就解决了。这个deepcopy函数，同事也是网上copy的，看来有些场景还是要分析清楚才能直接用。

不过最关键的还是性能测试工具没有及时的和游戏版本同步导致的。

