# 1.研究背景
 -   基于云计算和存储服务的兴起，有了数据持有性的研究
 -   优点：用户能以低成本获得海量的存储
 -   缺点：用户失去了对数据的实际控制权，数据面临着窥视，篡改，破坏等风险，数据完整性证明机制的出现及时识别破坏数据的行为，保证云数据的完整性。
 -   根据是否对数据文件采用容错处理，分为：
    PDP（Provale Date Possession）数据持有性证明；
    POR(Proofs of Retirevability) 数据可恢复证明

***********
# 2.数据持有性证明方案分类

 PDP分为以下5种方式：
- （1）MAC认证码：函数MAC<sub>sk</sub>(.) 为数据F生成验证元数据集合MAC={mac<sub>i</sub>}<sub>1<=i<=n</sub>
-  (2) 基于RSA签名：RSA签名机制的同态特性(a<sup>m</sup>)<sup>r</sup>=a<sup>rm</sup>=(a<sup>r</sup>)<sup>m</sup>(mod N)
- （3）基于BLS签名：e:G&times;G&rarr;G<sub>T</sub>双线性映射
- （4）支持动态操作：Merkle树，跳表等数据结构
- （5）支持多副本：利用冗余备份的方式来存储重要的文件
    
    
    
## 2.1 基于BLS签名的PDP方案
   e:G&times;G&rarr;G<sub>T</sub>双线性映射,g为G的生成元，H:{0,1}<sub>*</sub>&rarr;G为BLS哈希函数
- Setup阶段：
 - （1）随机选取私钥a&larr;z<sub>p</sub>，计算相应的公钥v=g<sup>n</sup> 
 - （2）选区唯一的文件标识v&larr;{0,1}<sup>k</sup>，选区随机数u&larr;G，对文件进行分块，F={b<sub>1</sub>,....,b<sub>n</sub>}生成元数据集合&theta;={&gamma;<sub>i</sub>}<sub>1<=i<=n</sub>其中&gamma;<sub>i</sub>=(h(v||i)&times;u<sup>m</sup><sub>i</sub>)<sup>a</sup>
 - (3) 将F和&theta;存入云
 - chanllenge &Verify阶段：
 - （1）TPA从数据快[1,n]中随机选c个块索引，为每个快索引i选随机数v<sub>i</sub>	
&larr;Z<sub>p/2</sub>chal={i,v<sub>i</sub>}<sub>s<sub>1</sub><=i<=s<sub>c</sub></sub>
- （2）云接受到chal后，计算u=&Sigma;<sup>s<sub>c</sub></sup><sub>i=s<sub>i</sub></sub>m<sub>i</sub>v<sub>i</sub>计算&gamma;将{&gamma;,u}返回
- （3）TPA验证：对攻击返回的结果与原先存储的结果进行对比，看结果是否一致，若是一致则可说明数据没有没纂改，若是不一致则说明数据发生了改变
 

## 2.2 PDP方案面临的问题
 - challenge随机挑战信息的生成依赖于TPA（相当于一个可信赖的第三方）,这个随机挑战值是否安全
 - 为了更高的检测率，对多块数据生成随机挑战，大大增加了发起挑战的复杂度
 - 交互式的挑战要求必须为强同步网络，交互次数过多造成系统网络负载增加
 - 储存矿工保存大量订单，如果每一次挑战针对一个文件，挑战数量会严重影响系统负载
 - 如果一份数据以多副本方式保存，如何解决女巫攻击和多副本的虚假保存
 - PDP可以证明当前时刻保存了文件，如何确保在两个检测周期之间是否储存了文件
 
## 2.3.1 解决TPA随机挑战问题
- 方案：Verifiable Random Function
- 理论依据：实质非对称密钥技术的哈希函数（常用椭圆曲线算法）
- 具体操作流程如下：
- （1）证明者生成一对密钥，PK,SK;
- （2）证明者计算rersult=VRF_Hash(SK,info);
- （3）证明者计算proof=VRF_Proof(SK,info);
- （4）证明者把result和proof递交给验证者；
- （5）验证者计算result=VRF_P2H(proof)是否成立，若成立，继续下面的步骤，否则终止；
- （6）证明者把PK,infO递交给验证者；
- （7）验证者计算True/False=VRF_Verify(PK,info,proof)

## 2.3.2 解决副本复制：时间证明问题
 - 在时间t内通过连续迭代多次证明，来实现在t时间内存储数据，但是会有一个问题就是：我们该如何保证在这个时间t内矿工没有删除数据？
 - 方案：Verifiable Delay Function(VDF)
 - VDF定义：
 - 延迟时间t,可验证延迟函数f
 - （1）连续性：t个步骤计算f(x)
 - （2）可高效验证：任何观察者可以在很短时间(log(t))内验证y=f(x)在达到最终结果之前，任何人都无法将f(x)的输出与其它随机数区分开
 - 简单VDF构造：(N,x,T) N=p &times;q ,y=x<sup>2<sup>T</sup></sup>mod N
 - 在不知道N的因式分解的前提下计算：x&rarr;x<sup>2</sup>&rarr;x<sup>2<sup>2</sup></sup>&rarr;x<sup>2<sup>3</sup></sup>&rarr;.....&rarr;x<sup>2<sup>T</sup></sup>mod N