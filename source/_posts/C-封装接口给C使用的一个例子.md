---
title: C++封装接口给C使用的一个例子
date: 2017-04-28 01:10:48
tags:
---
在编写模块代码时，我们希望模块既可以被C项目使用，也可以被C\++项目使用，下面是一种界面展示方式，在实现上使用C\++实现，api上提供两套类似的界面， 这种方式既提供了C的访问界面，也没把**[对象](https://en.wikipedia.org/wiki/Object-oriented_programming)**给弄丢了，代码摘自**[openh264](http://www.openh264.org/)**
<!-- more -->
### lib.h
头文件中通过 **__cplusplus** 来判断使用者是C还是C++
```c++
#ifdef __cplusplus

class IEncoder
{
public:
	virtual int Initialize () = 0;

	virtual int EncodeFrame ()= 0;

	virtual int SetOption ()= 0;

	virtual int GetOption ()= 0;

	virtual ~IEncoder(){};
};

extern "C"
{
#else

	typedef struct _IEncoder IEncoder_t;
	typedef const IEncoder_t* IEncoder;
	struct _IEncoder
	{

		int (*Initialize) (IEncoder*);

		int (*EncodeFrame) (IEncoder*);

		int (*SetOption) (IEncoder*);

		int (*GetOption) (IEncoder*);
	};

#endif
	typedef enum
	{
		IENCONDE_TYPE_H264,
		IENCONDE_TYPE_OPH264,
		IENCONDE_TYPE_X264,
		IENCONDE_TYPE_MAX
	}IENCONDE_TYPE_T;
	int  CreateEncoder (IEncoder** ppEncoder, IENCONDE_TYPE_T type);

#ifdef __cplusplus
}
#endif
```
### lib.cpp
 API 实现,C++语法
```c++
#include "lib.h"
#include "iostream"
using namespace std;


class H264Encoder  :public IEncoder
{
public:
	H264Encoder();
	H264Encoder(string type)
	{
		this->type = type;
	};
	virtual ~H264Encoder();
	virtual int Initialize ();

	virtual int EncodeFrame ();

	virtual int SetOption ();

	virtual int GetOption ();
private:
	string type;
};
H264Encoder::H264Encoder()
{
	type=__FUNCTION__;
}
H264Encoder::~H264Encoder()
{

};
int H264Encoder::Initialize ()
{
	cout<<this->type<<"::"<<__FUNCTION__<<endl;
}

int H264Encoder::EncodeFrame ()
{
	cout<<this->type<<"::"<<__FUNCTION__<<endl;
}

int H264Encoder::SetOption ()
{
	cout<<this->type<<"::"<<__FUNCTION__<<endl;
}

int H264Encoder::GetOption ()
{
	cout<<this->type<<"::"<<__FUNCTION__<<endl;
}
//****************************************************************************************//
class OPH264Encoder : public H264Encoder
{
public:
	OPH264Encoder (): H264Encoder("OPH264Encoder"){};
	virtual ~OPH264Encoder (){};
};
//****************************************************************************************//
class X264Encoder : public H264Encoder
{
public:
	X264Encoder (): H264Encoder("X264Encoder"){};
	virtual ~X264Encoder (){};
};



int  CreateEncoder (IEncoder** ppEncoder, IENCONDE_TYPE_T type)
{
	switch(type)
	{
		case  IENCONDE_TYPE_H264:
			*ppEncoder = new H264Encoder();
			break;
		case  IENCONDE_TYPE_OPH264:
			*ppEncoder = new OPH264Encoder();
			break;
		case  IENCONDE_TYPE_X264:
			*ppEncoder = new X264Encoder();
			break;
		default:
			*ppEncoder = NULL;
			break;
	}
	if ((*ppEncoder) != NULL)
	{
		return 0;
	}

	return 1;
}
```
### main.c
测试程序
```c
#include "c_api.h"
#include "stdio.h"
int main()
{
	int ii;
	IEncoder * enc;
	for(ii = IENCONDE_TYPE_H264; ii< IENCONDE_TYPE_MAX; ii++)
	{
		printf("\nCreateEncoder type=%d\n", ii);
		CreateEncoder(&enc, ii);
		(* enc)->Initialize(enc);
		(* enc)->EncodeFrame(enc);
		(* enc)->SetOption(enc);
		(* enc)->GetOption(enc);
	}
	return 0;
}
```
