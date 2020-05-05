สำหรับข้อนี้ เราสามารถสรุปโจทย์ได้ว่าโจทย์จะให้ต้นไม้ที่มีฉลากมาทั้งหมด $t$ ต้น ซึ่งต้นไม้ $t$ ต้นนั้นอาจมีซ้ำกันได้ (ซ้ำตามคุณสมบัติ) แล้วมีต้นไม้ที่มีฉลากอีก $q$ ต้นเพื่อมาตรวจสอบว่าแต่ละต้นที่ต้องการตรวจสอบนั้น มีอยู่ในฐานข้อมูลกี่ต้น

เช่น 

$t=2$


3

1 2

1 3

กับ

3

2 1

3 1


ในกรณีนี้ สองต้นไม้ที่มีฉลากนี้ถือว่าเหมือนกัน ดังนั้นถ้า $q=1$ และมีข้อมูลนำเข้าดังนี้

3

1 2

3 1


ในกรณีนี้จะให้คำตอบออกมาเป็น 2 เพราะมีต้นไม้ที่มีฉลากในฐานข้อมูลที่เหมือนกับต้นไม้ที่มีฉลากที่ให้มาเพื่อตรวจสอบรวมทั้งสิ้น 2 ต้น

ดังนั้นสิ่งที่เราจะต้องคิดคือเราจะทำให้โปรแกรมตรวจสอบต้นไม้ที่มีฉลากที่ในความเป็นจริงเหมือนกัน แต่การนำเข้ามาอาจไม่เหมือนกัน ให้มันเหมือนกันได้อย่างไร และถ้ามันเหมือนกัน เราจะเก็บจำนวนที่นับได้ไว้ตรงไหน

**ขั้นตอนที่ 0 : ทำการเก็บต้นไม้ที่มีฉลาก**

วิธีการเก็บนั้นมีหลายประเภทเช่น adjacency matrix, edge list อย่างไรก็ตาม ในวิธีทำนี้ ผู้เขียนจะเลือกใช้ edge list ซึ่งจะอธิบายเหตุผลในภายหลัง

**ขั้นตอนที่ 1 : ทำการจัดระเบียบข้อมูล**

สมมติว่าถ้าเรายกเส้นเชื่อมปมต้นไม้มา 2 เส้นคือ `1-2` กับ `2-1` เราจะพบว่าสองเส้นนี้เหมือนกันเพราะต้นไม้ที่โจทย์บอกมานั้น ทิศทางไม่มีผล ดังนั้นเราก็ทำการสลับเลขให้มันเหมือนกัน 

และนี่คือเหตุผลที่เลือกใช้ edge list เพราะเวลาเก็บกราฟในรูปแบบนี้มันจะอยู่ในรูปแบบ `[(1,2),(1,3)]` ถ้าข้อมูลเข้าถูกใส่มาแบบ `[(2,1),(3,1)]` เราสามารถสลับ `(2,1)` ให้เป็น `(1,2)` แล้วค่อยบรรจุข้อมูลลง `[]` ได้ ซึ่งอาจจะใช้คำสั่ง `swap()` ไม่ก็ทำการเขียนสลับด้วยตัวเอง

อย่างไรก็ตาม การสลับตัวเลขก็ไม่เพียงพอต่อการทำมาเช็คความเหมือนกันของต้นไม้เพราะอาจจะเกิดกรณีดังนี้ได้

`[(1,3),(1,2)]` กับ `[(1,2),(1,3)]`

ไม่ก็

`[(1,2),(3,4)]` กับ `[(3,4),(1,2)]`

ซึ่งในความเป็นจริง ทั้งสองกรณีนี้ ถ้านำมาเทียบความเหมือน ควรให้ผลออกมาเป็น `TRUE` ทั้งคู่ แต่ถ้านำไปเขียนจริง โปรแกรมจะให้ผล `FALSE` ทั้งคู่ 

ดังนั้นสิ่งที่เราต้องทำคือ**ทำการเรียงลำดับข้อมูล**อีกรอบเช่น

`[(1,3),(1,2)]` ไปเป็น `[(1,2),(1,3)]`

ซึ่งการทำงานนี้ เราสามารถใช้คำสั่ง `sort()` ในการช่วยเขียนได้

**ขั้นตอนที่ 2 : เก็บต้นไม้หลังจากการจัดระเบียบข้อมูลไว้ เพื่อรอการเรียกตอบในขั้นตอนต่อๆไป**

ในขั้นตอนนี้ เราจะเลือกโครงสร้างข้อมูลที่สามารถตั้งค่า index ด้วยตัวเองได้ 

array จะไม่เหมาะสมเท่าไหร่เพราะ index จะเป็นได้แค่ตัวเลข เช่น `arr[1]=2` 

ดังนั้น map จึงเป็นโครงสร้างข้อมูลหนึ่งที่เหมาะสมกับขั้นตอนนี้ โดยเราจะให้ข้อมูลตัวแรก(ทำหน้าที่เป็น index) เป็นต้นไม้ และข้อมูลตัวที่สอง เป็นตัวเลขที่นับว่ามีต้นไม้นั้นกี่ต้นแล้ว

ยกตัวอย่างการทำงาน


`map<...> mp`

ได้ input แรกเป็น `[(1,2),(1,3)]`

`mp[[(1,2),(1,3)]] = 1`

ได้ input ที่สองเป็น `[(2,1),(1,3)]`

`mp[[(1,2),(1,3)]] = 2`


**ขั้นตอนที่ 3 : นำต้นไม้ที่ต้องการตรวจสอบมาค้นหาจำนวนแล้วสั่งตอบ**

ในขั้นตอนที่ 2 เราได้ทำการตรวจสอบแล้วว่าในฐานข้อมูลนั้นๆ มีต้นไม้ประเภทเดียวกัน(หลังจัดระเบียบข้อมูล)กี่ต้น ซึ่งได้เก็บค่านั้นไว้ใน map เป็นที่เรียบร้อยแล้ว ทีนี้เราก็จะทำการรับต้นไม้ที่มีฉลากที่ต้องการตรวจสอบมาเช็คว่ามีต้นไม้ประเภทเดียวกับที่ต้องการตรวจสอบทั้งหมดกี่ต้นในระบบ 


ซึ่งวิธีการทำนั้นคือเราต้องเอาต้นไม้ที่รับมามา "จัดระเบียบ" ก่อนนำไปเช็คกับฐานข้อมูลที่มี โดยเขียนวิธีคิดโดยคร่าวได้ดังนี้


ได้ input มาเป็น `[(2,1),(3,1)]`

จัดระเบียบเป็น `[(1,2),(1,3)]`

แล้ว `print(mp[[(1,2),(1,3)]])`


Solution Code in C++
```cpp

#include "bits/stdc++.h"
using namespace std;
typedef pair<int,int> ii;
#define f first
#define s second
int n,m;
map<vector<ii>,int> M;
std::vector<ii> rt()
{
    int z;
    scanf("%d",&z);
    vector<ii> V;
    int u,v;
    for(int i=1;i<z;i++)
    {
        scanf("%d %d",&u,&v);
        if(u>v) swap(u,v);
        V.emplace_back(u,v);
    }
    sort(V.begin(),V.end());
    return V;
}
int main(int argc, char const *argv[])
{
    scanf("%d %d",&n,&m);
    while(n--) M[rt()]++;
    while(m--) printf("%d\n",M[rt()]);
}

```