---
layout: post
title: "Template_Builder_DesignPattern"
categories: [designpattern] 
tags: [C/C++]
---
模板方法模式 建造者模式 C++实现
================================
今天我继续来填坑，模板方法模式就是在模板方法中按照一定的规则顺序调用基本方法。这个比较简单。

比如我在父类run（）中调用类中一系列的方法。钩子是一种方法，它在抽象类中不做事，或者只做默认的事情，子类可以选择要不要去覆盖它。

___Java中，如果父类里面有些方法不允许覆写，那么为了防止子类改变模板方法中的算法，可以将模板方法声明为final。___

![](/assets/pic/IMG_0432.jpg)

<pre><code>
#include <iostream>
using namespace std;
 
class CarTemplate
{
public:
    CarTemplate(void){};
    virtual ~CarTemplate(void){};
 
protected:
    virtual void Start() = 0;
    virtual void Stop() = 0;
    virtual void Alarm() = 0;
    virtual void EngineBoom() = 0;
    virtual bool IsAlarm() =0;
 
public:
        void Run()
        {
        Start();
 
        EngineBoom();
 
        if(IsAlarm())
            Alarm();
 
        Stop();
        };
};
 
class Hummer: public CarTemplate
{
public:
    Hummer(){m_isAlarm = true;}
    virtual ~Hummer(){};
 
protected:
    void Start(){   cout<< "Hummer Start" << endl;   }
    void Stop(){    cout<< "Hummer Stop" << endl;    }
    void Alarm(){   cout<< "Hummer Alarm" << endl;}
    bool IsAlarm(){ return m_isAlarm;       }
    void EngineBoom(){  cout<< "Hummer EngineBoom" << endl;  }
private:
    bool m_isAlarm;
};
 
class Benz: public CarTemplate
{
public:
    Benz(){m_isAlarm = false;}
    virtual ~Benz(){};
 
protected:
    void Start(){   cout<< "Benz Start" << endl; }
    void Stop(){    cout<< "Benz Stop" << endl;  }
    void Alarm(){   cout<< "Benz Alarm" << endl;}
    bool IsAlarm(){ return m_isAlarm;       }
    void EngineBoom(){  cout<< "Benz EngineBoom" << endl;    } 
private:     
        bool m_isAlarm; 
}; 

int main() 
{   
        CarTemplate *hummer = new Hummer();     
        hummer->Run();
    delete  hummer;
 
    CarTemplate *benz = new Benz();
    benz->Run();
    delete benz;
}
</code></pre>

优点就在于可以封装不变的部分，拓展可变的部分，也就是把共性的东西抽取出来，这样可以便于维护。

但是缺点在于子类的执行，影响到父类的结果。有时候会有一些晦涩难懂。
———————————————————————————————-

***在模板方法中，我们把相同的都封装在父类中，不同都定义在子类中。但是需求的变化是无穷无尽的。***

> 比如建造的顺序可能是变化的，所以将他放到builder类中，通过builder类获取真正的product！

![](/assets/pic/6446jgG.gif)

<pre><code>
class Builder
{
public:
    virtual void BuildHead() {}
    virtual void BuildBody() {}
    virtual void BuildLeftArm(){}
    virtual void BuildRightArm() {}
    virtual void BuildLeftLeg() {}
    virtual void BuildRightLeg() {}
};
//构造瘦人
class ThinBuilder:public Builder
{
public:
    void BuildHead() { cout<< "build thin body" << endl; }
    void BuildBody() { cout<< "build thin head" << endl; }
    void BuildLeftArm() { cout<< "build thin leftarm" << endl; }
    void BuildRightArm() { cout<< "build thin rightarm" << endl; }
    void BuildLeftLeg() { cout<< "build thin leftleg" << endl; }
    void BuildRightLeg() { cout<< "build thin rightleg" << endl; }
};
//构造胖人
class FatBuilder : public Builder
{
public:
    void BuildHead() { cout<<"build fat body"<< endl; }
    void BuildBody() { cout<<"build fat head"<< endl; }
    void BuildLeftArm() { cout<<"build fat leftarm"<< endl; }
    void BuildRightArm() { cout<<"build fat rightarm"<< endl; }
    void BuildLeftLeg() { cout<<"build fat leftleg"<< endl; }
    void BuildRightLeg() { cout<<"build fat rightleg"<< endl; }   
};   //构造的指挥官   

class Director     {   
private:       
    Builder *m_pBuilder;   
public:       
    Director(Builder *builder) { m_pBuilder = builder; }       
    void Create()
    {           
        m_pBuilder->BuildHead();
        m_pBuilder->BuildBody();
        m_pBuilder->BuildLeftArm();
        m_pBuilder->BuildRightArm();
        m_pBuilder->BuildLeftLeg();
        m_pBuilder->BuildRightLeg();
    }
};
 
int main()
{
    FatBuilder fat;
    Director director(&fat);
    director.Create();
---------------------------------------------------------------------------
    Builder *thin = new ThinBuilder();
    Director *dir = new Director(thin);
    dir->Create();
 
    return 0;
}
</code></pre>

___建造者模式的优点：在于我们不用关心每个产品内部的组成细节，比如FatBuildier ThinBuilider，然后我们交给Director来构造就好了。___

建造者是独立的，我们可以拓展更多的builder完成不同的功能。并不对系统有任何影响。

___特别适合产品类特别复杂，顺序不同，效能不同。顺序是区别模板模式与建造者模式最大的不同点！__

> 另外同工厂模式进行比较，工厂模式适合生产零件。不能再细分了。否则就不适合工厂模式。

#####把模板方式和建造者模式组合起来可以产生很好的效果，可以按照model通过builder批量生产某种特定型号的产品。


![](/assets/pic/IMG_0433.jpg)

<pre><code>
#include<iostream>
#include<vector>
#include<string>
using namespace std;

class CCarModel
{
public:
    CCarModel(void) {}
    virtual ~CCarModel(void){}
    void Run()
    {
            vector::const_iterator it = m_pSequence->begin();
	    for (; it < m_pSequence->end(); ++it)
	    {
	        string actionName = *it;
	        if(actionName.compare("start") == 0)
	        {
	            Start();
	        }
	        else if(actionName.compare("stop") == 0)
	        {
	            Stop();
	        }
	        else if(actionName.compare("alarm") == 0)
	        {
	            Alarm();
	        }
	        else if(actionName.compare("engine boom") == 0)
	        {
	            EngineBoom();
	        }
	    }
    }
    void SetSequence(vector *pSeq)
    {
    		m_pSequence = pSeq;
    }
protected:
    virtual void Start() = 0;
    virtual void Stop() = 0;
    virtual void Alarm() = 0;
    virtual void EngineBoom() = 0;
private:
    vector * m_pSequence;
};

class CBenzModel : public CCarModel
{
public:
    CBenzModel(void){}
    ~CBenzModel(void){}
protected:
    void Start(){cout << "Benz Start..." << endl;}
    void Stop(){cout << "Benz Stop..." << endl;}
    void Alarm(){cout << "Benz Alarm" << endl;}
    void EngineBoom(){cout << "Benz EngineBoom...." << endl;}
};

class CBMWModel : public CCarModel
{
public:
    CBMWModel(void){}
    ~CBMWModel(void){}
protected:
    void Start(){cout << "BMW Start..." << endl;}
    void Stop(){cout << "BMW Stop..." << endl;}
    void Alarm(){cout << "BMW Alarm" << endl;}
    void EngineBoom(){cout << "BMW EngineBoom...." << endl;}
};

class ICarBuilder
{
public:
    ICarBuilder(void)   { }
    virtual ~ICarBuilder(void)  { }
    virtual void SetSequence(vector *pseq) = 0;
    virtual CCarModel * GetCarModel() = 0;
};

class CBenzBuilder : public ICarBuilder
{
public:
    CBenzBuilder(void);
    ~CBenzBuilder(void);
    void SetSequence(vector *pSeq);
    CCarModel * GetCarModel();
private:
    CCarModel *m_pBenz;
};
CBenzBuilder::CBenzBuilder(void)
{
    m_pBenz = new CBenzModel();
}
CBenzBuilder::~CBenzBuilder(void)
{
    delete m_pBenz;
}
void CBenzBuilder::SetSequence(vector *pSeq)
{
    m_pBenz->SetSequence(pSeq);
}
CCarModel * CBenzBuilder::GetCarModel()
{
    return m_pBenz;
}

class CBMWBuilder :public ICarBuilder
{
public:
    CBMWBuilder(void);
    ~CBMWBuilder(void);
    void SetSequence(vector *pSeq);
    CCarModel * GetCarModel();
private:
    CCarModel *m_pBMW;
};

CBMWBuilder::CBMWBuilder(void)
{
    m_pBMW = new CBMWModel();
}
CBMWBuilder::~CBMWBuilder(void)
{
    delete m_pBMW;
}
void CBMWBuilder::SetSequence( vector *pSeq )
{
    m_pBMW->SetSequence(pSeq);
}
CCarModel * CBMWBuilder::GetCarModel()
{
    return m_pBMW;
}

class CDirector
{
public:
    CDirector(void);
    ~CDirector(void);
    CBenzModel * GetABenzModel();
    CBenzModel * GetBBenzModel();
    CBMWModel * GetCBMWModel();
    CBMWModel * GetDBMWModel();
private:
    vector * m_pSeqence;
    CBenzBuilder * m_pBenzBuilder;
    CBMWBuilder * m_pBMWBuilder;
};

CDirector::CDirector(void)
{
    m_pBenzBuilder = new CBenzBuilder();
    m_pBMWBuilder = new CBMWBuilder();
    m_pSeqence = new vector();
}
CDirector::~CDirector(void)
{
    delete m_pBenzBuilder;
    delete m_pBMWBuilder;
    delete m_pSeqence;
}
CBenzModel * CDirector::GetABenzModel()
{
    m_pSeqence->clear();
    m_pSeqence->push_back("start");
    m_pSeqence->push_back("stop");
    m_pBenzBuilder->SetSequence(m_pSeqence);
    return dynamic_cast<CBenzModel*>(m_pBenzBuilder->GetCarModel());
}
CBenzModel * CDirector::GetBBenzModel()
{
    m_pSeqence->clear();
    m_pSeqence->push_back("engine boom");
    m_pSeqence->push_back("start");
    m_pSeqence->push_back("stop");
    m_pBenzBuilder->SetSequence(m_pSeqence);
    return dynamic_cast<CBenzModel*>(m_pBenzBuilder->GetCarModel());
}
CBMWModel * CDirector::GetCBMWModel()
{
    m_pSeqence->clear();
    m_pSeqence->push_back("alarm");
    m_pSeqence->push_back("start");
    m_pSeqence->push_back("stop");
    m_pBMWBuilder->SetSequence(m_pSeqence);
    return static_cast<CBMWModel*>(m_pBMWBuilder->GetCarModel());
}
CBMWModel * CDirector::GetDBMWModel()
{
    m_pSeqence->clear();
    m_pSeqence->push_back("start");
    m_pBenzBuilder->SetSequence(m_pSeqence);
    return dynamic_cast<CBMWModel*>(m_pBMWBuilder->GetCarModel());
}
int main(int argc, char const *argv[])
{
	CDirector *dir = new CDirector();

	dir->GetBBenzModel()->Run();
	return 0;
}


</code></pre>

通过上面可以看出两个模式的配合得到了很好的运用！

引用 http://blog.csdn.net/wuzhekai1985/article/details/6667467