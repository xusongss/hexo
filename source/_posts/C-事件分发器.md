---
title: C++事件分发器
date: 2017-05-12 16:05:27
tags:
---
# event
xevent.h
```c++
class XEvent
{
public:
	virtual void dump() = 0;
	virtual void free() = 0;
};
```
<!--more-->
# EventListener
xeventlistener.h
```c++
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
```c++
#include <list>
class XEvent;
class XEventListener;
class XEventDispatcher
{
public:
	//添加监听器
	virtual void addEventListener(XEventListener* listener, int fixedPriority = -1) ;

	//删除指定监听器
	virtual void removeEventListener(XEventListener* listener);

	//启用事件分发器
	virtual void setEnabled(bool isEnabled) ;

	virtual bool isEnabled() const ;
	//手动派发自定义事件
	virtual void dispatchEvent(XEvent* event) ;
};
```
xeventdispatcher.cpp

```c++
#include "xeventdispatcher.h"
#include "xevent.h"
#include "xeventlistener.h"
XEventDispatcher::XEventDispatcher():
mEnable(false)
{
}


XEventDispatcher::~XEventDispatcher()
{
}

//添加监听器
void XEventDispatcher::addEventListener(XEventListener* listener, int fixedPriority){
	mls.push_back(listener);
}
//删除指定监听器
void XEventDispatcher::removeEventListener(XEventListener* listener){
	mls.remove(listener);
}
//启用事件分发器
void XEventDispatcher::setEnabled(bool isEnabled){
	mEnable = isEnabled;
}
bool XEventDispatcher::isEnabled() const{
	return mEnable;
}
//手动派发自定义事件
void XEventDispatcher::dispatchEvent(XEvent* event){
	if (mEnable){
		T_ListenerList::iterator it;
		for (it = mls.begin(); it != mls.end(); ++it){
			(*it)->onEvent(event);
		}
	}
}
```
# Factory
factory.h
```C
#pragma once
class XEventDispatcher;
class Factory
{
public:
	static XEventDispatcher * getEventDispatcher();
private:
	static XEventDispatcher * mXEventDispatcher;
};
```
factory.cpp
```c++
#include "factory.h"
#include "xeventdispatcher.h"

XEventDispatcher * Factory::mXEventDispatcher = NULL;
XEventDispatcher * Factory::getEventDispatcher(){
	if (!mXEventDispatcher){
		mXEventDispatcher = new XEventDispatcher;
	}
	return mXEventDispatcher;
}

```
# test
main.cpp
```c++
#include <iostream>
#include "xevent.h"
#include "xeventlistener.h"
#include "xeventdispatcher.h"
#include "factory.h"
class Mouse :virtual public XEvent{
	virtual void dump(){
		printf("Mouse event \r\n");
	}
};
class Keyboard :virtual public XEvent{
	virtual void dump(){
		printf("Keyboard event \r\n");
	}
};

class HandlerClass
{
public:
	int OnEvent(XEvent *evt)
	{
		evt->dump();
		return 0;
	}
};
int main(int argc, _TCHAR* argv[])
{
	HandlerClass h;
	Keyboard k1;
	Keyboard k2;
	Mouse m1;
	Mouse m2;
	{
		ClassEventListener<HandlerClass, Keyboard> mouseEvthandler(&h, &HandlerClass::OnEvent);
		ClassEventListener<HandlerClass, Mouse> keyboardEvthandler(&h, &HandlerClass::OnEvent);
		XEventDispatcher * dispatcher = Factory::getEventDispatcher();
		dispatcher->addEventListener(&mouseEvthandler);
		dispatcher->addEventListener(&keyboardEvthandler);
		dispatcher->dispatchEvent(&m1);
		dispatcher->dispatchEvent(&m2);
		dispatcher->dispatchEvent(&k1);
		dispatcher->dispatchEvent(&k2);
		dispatcher->setEnabled(true);
		dispatcher->dispatchEvent(&m1);
		dispatcher->dispatchEvent(&m2);
		dispatcher->dispatchEvent(&k1);
		dispatcher->dispatchEvent(&k2);

		dispatcher->removeEventListener(&mouseEvthandler);
		dispatcher->removeEventListener(&keyboardEvthandler);
	}
	printf("===================================================== \r\n");
	{
		ClassEventListener<HandlerClass> mouseEvthandler1(&h, &HandlerClass::OnEvent);
		ClassEventListener<HandlerClass> keyboardEvthandler1(&h, &HandlerClass::OnEvent);
		XEventDispatcher * dispatcher = Factory::getEventDispatcher();
		dispatcher->addEventListener(&mouseEvthandler1);
		dispatcher->addEventListener(&keyboardEvthandler1);
		dispatcher->dispatchEvent(&m1);
		dispatcher->dispatchEvent(&m2);
		dispatcher->dispatchEvent(&k1);
		dispatcher->dispatchEvent(&k2);
		dispatcher->removeEventListener(&mouseEvthandler1);
		dispatcher->removeEventListener(&keyboardEvthandler1);
	}

	getchar();
	return 0;
}
```
