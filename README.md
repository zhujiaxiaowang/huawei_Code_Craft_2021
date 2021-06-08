# 2021华为软件精英挑战赛 复赛方案
**队伍: 重大干饭人         
  初赛rank10         
  复赛rank7** 
# 赛题思路
## 一、服务器购买策略
由于评价策略的指标是整个时间段内的消耗成本，主要包括购买服务器时的成本和部署服务器每天的能耗成本两部分组成。几乎都能想到的是：**在前期应该主要考虑能耗成本，越到后期就应该越优先考虑购买成本**，思路很明确，重点是该如何权衡这两部分之间的比重，以及服务器不同资源比例对其又有何影响。 同类服务器在不同时期购买的性价比是动态变化的，为了更准确衡量服务器在每天的性价比，通过数学模型分析，我们提出了基于滑动窗口的有效资源利用比在每次需要购买新的服务器时来实时更新不同种类的服务器性价比。这一改进也是复赛中最大的提分点。总成本一下少了好几千万。代码实现如下：
```
int cpu = 0, mem = 0, count = 18;

for(int k = OpCntofOneDay; k < OneDayAdd_VC.size() && k < OpCntofOneDay+count; k++)
            {
                cpu += OneDayAdd_VC[k].CpuNeed;
                mem += OneDayAdd_VC[k].Memory_Need;
            }
            double rad  = 1.0 * cpu / mem;
            double rad1 = 1.0 * mem / cpu;
            for(auto &k : HostCate)
            {
                int value = min(k.CoreNum / rad + k.CoreNum, k.Memory / rad1 + k.Memory);
                k.myPrice = (1.0 * k.Prices * 1.01/ (DaysNum - Day) + k.EleCharge * 0.8) / (value+(k.CoreNum+k.Memory-value)*0.28);
                if(CpuNeedAll > MemNeedAll && k.CoreNum < k.Memory)
                    k.myPrice = 9999999;
                if(CpuNeedAll < MemNeedAll && k.CoreNum > k.Memory)
                    k.myPrice = 9999999;
            }
```
1.k.Prices表示当前时刻服务器种类k的性价比。可以将上面的公式拆解为分子和分母两部分： 第一部分[1.0 * k.Prices * 1.01/ (DaysNum - Day) + k.EleCharge * 0.8 ]代表当前购买该类服务器均摊到每天的成本,1.01和0.8分别代表后期平均购买成本和能耗成本的权值(这里假设购买的服务器在后期始终都处于开机状态，虽然现实很难实现，但这样的假设对大部分数据集都聚有良好的鲁棒性），其中k.Prices * 1.01/ (DaysNum - Day) 会随着Day的变化而变化，即越往后购买服务器的成本均摊到剩余每天的也就越大。
第二部分[value+(k.CoreNum+k.Memory-value)*0.28] 代表当前第k类服务器的真实有效资源利用个数。它是由前面计算出的有效资源利用个数加上一个补偿因子（补偿因子理解为：若服务器总共有60个资源，有效资源个数为40，那么剩下的20个资源后期仍可能通过迁移操作也被用到从而变为有效资源） 最后用当前的成本/真实有效利用资源个数，即得到服务器k当前时刻的性价比，在购买的时候对依据当前时刻性价比对所有种类服务器进行排序，购买第一个能放下最新虚拟机的服务器。 
PS:另外最后的两个if语句起始可以省滤掉，它是对不满足最小放置需求的服务器进行剔除，后面排序后也会再筛选一次

