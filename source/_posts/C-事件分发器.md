---
title: C++事件分发器
date: 2017-05-12 16:05:27
tags:
---
# event
xevent.h
```
class XEvent
{
public:
	virtual void dump() = 0;
	virtual void free() = 0;
};
```
# EventListener
xeventlistener.h
```
class XEvent;
class XEventListener {
public:
	virtual int onEvent(XEvent *evt) = 0;
};

template<class Handler, class filter = XEvent>
class ClassEventListener : virtual public XEventListener
{
public:
	typedef int (Handler::*__XHandlerFunc)(XEvent *evt);
	ClassEventListener(Handler *ptr, __XHandlerFunc func) :
		handler(ptr),
		func_(func){
	}

	virtual int onEvent(XEvent *evt)
	{
		if (dynamic_cast<filter*>(evt)){
			if (handler && func_)
				return (handler->*func_)(evt);
		}

		return 0;
	}
	~ClassEventListener(){};
private:
	Handler *handler;
	__XHandlerFunc func_;
};
```
# EventDispatcher
xeventdispatcher.h
```
#include <list>
class XEvent;
class XEventListener;
class XEventDispatcher
{
public:
	//添加监听器
	virtual void addEventListener(XEventListener* listener, int fixedPriority = -1) = 0;

	//删除指定监听器
	virtual void removeEventListener(XEventListener* listener) = 0;

	//启用事件分发器
	virtual void setEnabled(bool isEnabled) = 0;

	virtual bool isEnabled() const = 0;
	//手动派发自定义事件
	virtual void dispatchEvent(XEvent* event) = 0;
};
```
# test
main.cpp
```
```
