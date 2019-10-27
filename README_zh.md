# TNJ

本意是猫和老鼠。

## 题目构思

这题我其实构思了很久，可惜最后因为种种原因效果不佳，其中也有我的一些原因吧，所以也表示抱歉，我的经验不足导致题目没有达到我的预期效果。

这道题的思路来源是曾经 Atum 给我提过在 SECCON 玩过的一个游戏，由双方轮流执行汇编指令来玩，我本身对游戏挺感兴趣的，后来经过一些查找，搜到了这个游戏，叫做 [Core War](https://en.wikipedia.org/wiki/Core_War)。其实这个游戏在国内知名度并不高，本来我是打算直接使用的，但是这个游戏非常不妙，他有一个设定是平局。因为我们没有 KOH 的条件，所以一定存在攻防双方，平局判攻击方胜利或者防守方胜利都会导致大家更愿意平局（更加简单），于是我舍弃了这个方案，但是保留了这种执行汇编的思路。

之后我又思考了一些变体，经过一些搜索，找到了另一款游戏，叫做 [Darwin](https://corewar.co.uk/darwin.htm)，双方通过复制自身来玩，谁活下来谁赢。但是这个游戏也有一些问题，主要是攻击方和防守方的高度一致，因为在攻防环境中，要求攻击方和防守方非对称，否则攻击同时也是防御，不然就是分数乘2了，所以在此基础上我设计了这个游戏。

游戏的大致思路是结合两者的优点，我很喜欢 Core War 轮流执行汇编的场景，带来了很多变数，然后 Darwin 提供的游戏机制保证了游戏可以在规定时间内结束。所以，游戏被设计为双方轮流执行汇编，攻击方尝试驻留在程序中，防守方需要去破坏攻击方。

事实上这个过程还存在问题，比如攻击方如果一开始位置被防守方知道了，防守方可以直接破坏该位置，所以我决定让防守方无法获取攻击方加载位置，但是这样我又担心攻击方过强，所以加入了防守方可以看到攻击方输入代码的设定，这里也提供了代码分析的可能，也就是可以通过分析攻击方代码来应对。

在最开始的版本中，我打算使用动态参数调整，通过攻击方防守方的分数比例来决定一些参数，从而实现弱者保护，但是最终由于和平台方匹配上存在困难，只好作罢。

这其实表达了我对 AD 模式的一个理解， AD 模式其实并不像真正的安全环境的攻防，而更像是我们平时玩的游戏，在一道题目上体现 AD 的环境，要想激烈，有看点，其实最好的就是游戏。

但是同时这种游戏也不能纯粹是游戏，我本来是打算考大家的游戏策略来设计游戏，但是考虑到 CTF 需要的一些主题，最终没有这么去做。

## 运行过程

这道题目最终还是失败了，有几个原因吧。

第一个是我的提示不够，大家一开始没有理解我的意图，例如在打补丁的时候都是在我原来程序上去直接修改，而非自己编译，因为一开始的防守程序就有问题，所以直接能拿到 flag 也让选手表示难以理解，甚至没有动力再去理解了。

另外就是中途先因为和平台兼容时 checker 的 timeout 的问题，导致 checker 误判死循环，然后是因为缺少 alarm 导致两次资源耗尽，甚至不得不下线处理，这是实现上的问题，我对此感到很抱歉，也是我这次最大的遗憾，想了很多去设计，然后实现却做得不太好。

另一方面，可能也是由于我的提示不够强，到了第二天，我发现大家的 patch 都没有明显改进，我以为第一天由于攻击收益不如防守收益，大家都会多去考虑如何防守，但是第二天效果确实不佳，全场无法防守也会导致选手失去动力去维护这个题目，于是在一个小时之后选择了下线，辛苦了第一天晚上想到了新 payload 的同学哈哈。

## 攻守设计

这是我一开始预计的攻守设计：

1. 由于防守方为空，攻击方占优，可以很快出现 exp （我预计是半个小时，时间差不多在 40 多分钟出现一血，还算是达到预期）
2. 大家都看完题了，防守方知道修补方法，发现防守位置需要改一下（默认 patch 的问题是 jmp target 的问题，改改就好），通过 patch 快速修复，可以暂时避免无 shellcode 情况下的攻击。
3. 这个时候可以写攻击了，实现最基本的死循环，防守方还处于死循环阶段，于是攻击方占优，再次难以防守。
4. 防守方进化，循环覆盖整个 arena ，可以限制死循环攻击方。
5. 攻击方进化，通过各类复制自身方法攻击。
6. 防守方进化，通过分析攻击方程序进行阻止。
7. ...


在运行过程中，最终只有 1,2,3,4 部分完成了，5 和 6 都没有出现，第二天的攻击方进化最好的是我看到的一个没有预料到的 payload ，介于 3 和 4 之间，就是通过相对跳转完成死循环，但是事实上是无法攻破4阶段的。

5 阶段由于 m68k 架构存在复制连续寄存器到内存的指令，所以复制自身是比较靠谱的，但是最终都会有直接跳转，所以最终 6 可以防住这样的情况。

## 6 阶段示例

在比赛过程中，我也编写了 6 阶段的防守，可以供大家参考，我采用的方法是覆盖程序中出现的跳转目标，这里有一个问题，就是如果攻击方等一段时间再复制，那可能无法起到效果，不过这就很方便修改了，直接也等一段时间就好。

在第二天的测试中，我尝试了多个 payload ，都无法攻击这个防御，其中最接近攻破的是相对跳转的 payload，因为我只判断了 JMP 的目标，但是那个相对跳转死在了 tail 部分的覆盖中。

```C
#include <stdio.h>
#include <unistd.h>

struct shellcode {
    unsigned int size;
    unsigned int loc;
    char *code;
};

char clear_at[] = "\x20\x7c\x00\x00\x00\x00\x20\xfc\x00\x00\x00\x00";
char defender_shellcode[0x200];

char tail_shellcode[] = "\x20\x7c\x00\x00\x00\x00\x20\xfc\x00\x00\x00\x00\x4e\xf9\x00\x80\x00\x06";

void gen_defender(int *pipe, struct shellcode* _attacker) {
    int size = 0;
    for (unsigned int i = 0; i < _attacker->size - 3; i++) {
        if (_attacker->code[i] == (char)0x4e && _attacker->code[i + 1] == (char)0xf8) {
            // jmp xxxx
            clear_at[4] = _attacker->code[i + 2];
            clear_at[5] = _attacker->code[i + 3];
            for (int j = 0; j < 12; ++j) {
                defender_shellcode[size++] = clear_at[j];
            }
        }
    }
    for (int i = 0; i < 18; ++i) {
        defender_shellcode[size++] = tail_shellcode[i];
    }
    write(pipe[1], &size, sizeof(int));

    write(pipe[1], defender_shellcode, size);
}
```

## 总结

这次比赛我也学到了很多，一方面是对题目的把控，我的引导性不够强，导致发展一度没有往我所预期的方向，另外一个忽略的部分就是这种交互提升的设计，应该需要加入明牌设计，双方能够部分获取到对方的信息，例如防守方的信息可以公开，这样加速针对性进化。

其余就是实现上的一些细节了，这次由于测试不充分出了很多问题，这个就是经验了，下次应该就不会有了（不过或许下次我就不再写 AD 了。。。太容易出问题了）。

最后希望各位高抬贵手吧，为了大家能够有玩的感觉，我花了好长时间去搜索资料寻找思路来设计，虽然最后表现不佳。。我确实是比较菜，导致了这些问题，可能下次我这种菜鸡就只负责给个思路比较好了哈哈哈。