สำหรับข้อนี้ วิธีการตรงๆที่สามารถคิดได้ไม่ยาก คือดังนี้

เริ่มด้วยบิตสตริง $S$ (สำหรับข้อนี้ จะถือว่าเริ่มต้นด้วย index $0$ จนจบที่ $N-1$)

เราจะสร้างฟังก์ชัน $f$ นิยามโดย

```cpp
void f(string& S){
    S.push_back(S.front());
    S.erase(S.begin());
}
```

แล้วสามารถไล่ว่าทำฟังก์ชัน $f$ ซ้ำกี่ครั้งจึงจะได้ $S$ เหมือนเดิม ดังนี้

```cpp
string S;
string now = S;
for(int rep = 1; rep <= N; rep++){
    f(now);
    if(now == S){
        // Answer is rep.
        break;
    }
}
```

ซึ่งกระบวนการดังกล่าวใช้เวลา $\mathcal{O}(N^2)$ เพราะฟังก์ชัน $f$ และการเทียบสตริงจะใช้เวลา $\mathcal{O}(N)$ และมีการทำซ้ำทั้งหมด $N$ รอบ

อีกหนึ่งวิธีในการ optimize เบื้องต้น คือการเปลี่ยนการเก็บบิตสตริงจาก string เป็น bitset ซึ่งจะทำให้เร็วขึ้น 8 เท่า แต่ยังคง $\mathcal{O}(N^2)$ อยู่ จึงไม่ช่วยอะไรมาก

ต่อมาจะเป็นวิธีการ optimize เพื่อทำเวลาให้ดีขึ้นอย่างมีนัยสำคัญ โดยวิธีหลักๆมีอยู่ 2 วิธี

1. ใช้ข้อสังเกตบางอย่างทำให้เราไม่จำเป็นต้องเทียบสตริงทุก rep
2. ใช้ความรู้เรื่อง rolling hash เพื่อลดเวลาการทำ $f$ และ การเทียบสตริง

### วิธีที่ 1

สังเกตว่าคำตอบที่ออกมา จะเป็นตัวประกอบ (factor) ของ $N$ เท่านั้น วิธีการหนึ่งที่เป็นไปได้คือการหาตัวประกอบเตรียมรอไว้ก่อน แล้วเปลี่ยนนิยามฟังก์ชัน $f$ เพื่อให้ทำงานได้เร็วขึ้น

เราสร้าง $SS$ นิยามโดยนำสตริง $S$ มาต่อกันสองครั้ง (เช่น `S = "101110"` จะได้ `SS = "101110101110"`)

สังเกตว่าหากต้องการทำฟังก์ชัน $f$ ไป $K$ รอบ จะได้ผลลัพธ์เป็นส่วนตัดของสตริง $SS$ ในช่วง $K$ ถึง $K+N-1$

สมมติเราเก็บตัวประกอบทุกตัวของ $N$ ไว้ในเซต (จริงๆคือเวกเตอร์) $D$ แล้ว (ตัวอย่างโค้ดด้านล่าง)

```cpp
int N;
vector<int> D;
for(int i = 1; i <= N; i++){
    if(N % i == 0) D.push_back(i);
}
```

ต่อมาเราจะต้องไล่เช็คทุกส่วนตัดบนตัวประกอบแต่ละตัวว่าเหมือนกับ S หรือไม่ หากเหมือนกันให้นำตัวประกอบน้อยสุดมาคิด

```cpp
string S;
string SS = S+S;
for(int factor : D){
    bool ok = true;
    for(int i = 0; i < N; i++){
        if(SS[factor+i] == S[i]){
            // Everything is OK
        }else{
            // Mismatch!
            ok = false;
            break;
        }
    }
    if(ok){
        // Answer is factor.
        break;
    }
}
```

จากการกระทำข้างต้น เราจะได้ Time Complexity เป็น $\mathcal{O}(ND)$ เมื่อ $D$ แทนจำนวนตัวประกอบของ $N$ (ซึ่งโดยทั่วไปแล้วมีค่าน้อย สามารถพิสูจน์ได้โดยง่ายว่า $D \leq 2\sqrt{N}$ จากการสังเกตว่าหาก $D \vert N$ แล้ว $\frac{N}{D} \vert N$ ด้วย)

### วิธีที่ 2

ในวิธีนี้เราจะดัดแปลง *Rabin-Karp Algorithm* โดยจากเดิมที่เป็นการเทียบหา pattern ยาว $M$ ใน text ยาว $N$ ด้วยเวลา $\mathcal{O}(N+M)$ เราจะใช้ *Rolling Hash* เพื่อเปลี่ยนสถานะสตริงตามการหมุนของสวิทช์เวลา สมมติเรามี Hash function $h: \{0,1\}^{N} \rightarrow \mathbb{N}$ นิยามโดย $h(T) := \sum\limits_{i=0}^{N-1}{T_iB^{N-1-i}}$ สำหรับบางจำนวนนับ $B$ เราสามารถเทียบว่าสองสตริงเท่ากันได้หรือไม่โดยการเทียบว่าเมื่อผ่าน h ไปแล้ว จะเท่ากันหรือไม่ เพราะ $S = T \leftrightarrow h(S) = h(T)$ ปัญหาคือ $h(T) \in \mathcal{O}(B^{|T|})$ ทำให้อาจเกินขอบเขตของ `int` หรือ `long long` ได้ โดยจะมีวิธีแก้ง่ายๆเช่น ปล่อยเอาไว้อย่างนั้น เพราะ หากจำนวน overflow ไปเท่าไร มันจะ overflow ไปเท่ากัน หรืออีกวิธีคือนำไป $\mod p$ สำหรับบางจำนวนนับ $p$ (นิยมจำนวนเฉพาะ) แต่จะทำให้ได้ว่า $S = T \leftarrow h(S) = h(T)$ ไม่เป็นสัจนิรันดร์อีกต่อไป (มีกรณีที่อาจซ้ำได้) แต่เราสามารถคาดการณ์ได้ว่ามีโอกาสน้อยมากที่จะซ้ำกันเกิดขึ้น จึงสามารถใช้ต่อได้ อัลกอริทึมดังกล่าวจึงเป็น Monte Carlo Algorithm (หากกังวล สามารถใช้ double hash เพื่อลดโอกาสซ้ำ หรือ brute force ทุกครั้งที่พบว่าเท่ากัน เพื่อแปลงเป็น Las Vegas Algorithm)

ข้อดีของการทำ *Rolling Hash* คือเมื่อเราต้องการเคลื่อนย้ายตัวหน้าสุดไปไว้หลังสุด เราสามารถนำ $h(T)$ มาลบออกด้วย $T_0B^{N-1}$ แล้วคูณด้วย $B$ แล้วค่อยบวกกลับด้วย $T_0$ กล่าวคือ $h(f(T)) := B(h(T)-T_0B^{N-1})+T_0$ ได้เลย โดยไม่ต้องคำนวณ $h(f(T))$ ใหม่ตั้งแต่ต้น

เราทำการคำนวณ $h(f(S))$, $h(f(f(S)))$, $\dots$ จนกว่าจะพบว่า $h(f(f(\dots f(S)))) = h(S)$

การคำนวณ $h(T)$ แต่ละรอบ ใช้เวลา $\mathcal{O}(T)$ แต่หากมี $h(T)$ อยู่แล้ว สามารถหา $h(f(T))$ ได้ใน $\mathcal{O}(1)$ ตามวิธีการข้างต้น สุดท้ายจึงใช้เวลารวม $\mathcal{O}(N)$

โค้ดตัวอย่าง

```cpp
const int B = 53; // Select some random rolling hash base
int h(string T){
    int ret = 0;
    for(int i = 0; i < T.size(); i++){
        ret *= B; // Multiply by base
        ret += T[i] - '0'; // Convert char '0','1' to integer 0, 1.
        // Let ret overflow...
    }
    return ret;
}
string S;
string SS = S+S;
int N = S.size();
const int target = h(S);
int currentHash = target;
int highestMultiplier = 1;
for(int i = 0; i < N-1; i++) highestMultiplier *= B;
for(int i = 0; i < N; i++){
    currentHash -= (S[i] - '0') * highestMultiplier;
    currentHash *= B;
    currentHash += S[i] - '0';
    if(currentHash == target){
        // Answer should be i+1
        // Recheck for assurance
        bool correct = true;
        for(int j = 0; j < N; j++){
            if(SS[i+1+j] == S[j]){
                // Still OK
            }else{
                correct = false;
                break;
            }
        }
        if(correct){
            // Answer is exactly i+1
        	break;
        }
    }
}
```

