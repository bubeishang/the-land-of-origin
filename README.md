#include<iostream>
#include<cstdio>//printf
using namespace std;

class Motor {
public:
    virtual void Setoutput(float power) = 0;
    virtual float Getcurrent() = 0;
    virtual void Update() = 0;
    virtual ~Motor() {}
};//抽象基类

class DJImotor : public Motor {
public:
    void Setoutput(float power) override {
        printf("[CAN BUS] ID:0x201, Send Current:%.0f\n", power);
    }
    float Getcurrent() override {
        return 1000.0;
    }
    void Update() override {
 //没改变速度，返回值也是固定的，所以不需要更新
    }
};//DJI硬件电机

class SimMotor : public Motor {
    float velocity_ = 0.0;
    float inputpower = 0.0;
    float resistance = 0.1; 
public:
    void Setoutput(float power) override {
        inputpower = power;
    }
    void Setresistance(float a) {
        resistance = a;//阻力值也可以自己定义
    }
    float Getcurrent() override {
        return velocity_;
    }
    void Update() override {
        velocity_ = velocity_ + (inputpower - resistance * velocity_);//更新速度
    }
};//物理仿真电机

void Testmotor(Motor* motor) {
    float speed = 0.0;
    for (int i = 0; i < 5; i++) {
        motor->Setoutput(100.0 * (i + 1));  
        motor->Update();
        speed = motor->Getcurrent();
        cout << "当前速度: " << speed <<endl;  
    }

    cout << "最终速度: " << speed <<  endl;
}//测试函数

int main() {
   //测试DJI
    Motor* djiMotor = new DJImotor();
    Testmotor(djiMotor);
    delete djiMotor;
    //测试sim
    Motor* simMotor = new SimMotor();
    Testmotor(simMotor);
    delete simMotor;

    return 0;
}
