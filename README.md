# XiaWorld
了不起的修仙模拟器MOD教程

## 快速画符相关的修改

### 快速画符品质100%

**【修改的类】** Wnd_Painter

**【原版代码】** 
```csharp

````

**【修改内容】** 
```csharp
private void QuickP()
	{
		if (this.UIInfo.m_n62.grayed)
		{
			return;
		}
		if (this.CallBack != null)
		{
			this.CallBack(this.SelectName, 1f, null, false);//直接改成1f,固定100%画符品质
		}
		base.Hide();
	}
````
### 跳过第一次手动画符

**【修改的类】** Wnd_Painter

**【原版代码】** 
````
图片
````

**【修改内容】** 
```csharp
private void OnSelectGong(EventContext context)
	{
		string text = (string)(context.data as GObject).data;
		if (this.SelectName == text)
		{
			return;
		}
		this.SelectName = text;
		if (string.IsNullOrEmpty(text))
		{
			MapRender.Instance.PainRender.sharedMaterial.SetTexture("_Temp", null);
			this.UIInfo.m_n62.grayed = true;
			this.UIInfo.m_n63.text = null;
		}
		else
		{
			SpellDef spellDef = PracticeMgr.Instance.GetSpellDef(text);
			MapRender.Instance.PainRender.sharedMaterial.SetTexture("_Temp", Resources.Load<Texture2D>(spellDef.Template));
			float num = World.Instance.GetFuValue(text);
			num = 1f;//固定获取的画符值为100%，跳过第一次为0的情况。
			if (num > 0f)
			{
				this.UIInfo.m_n62.grayed = false;
				this.UIInfo.m_n63.text = string.Format("{0:P0}", num);
			}
			else
			{
				this.UIInfo.m_n62.grayed = true;
				this.UIInfo.m_n63.text = null;
			}
		}
		MapRender.Instance.PaintPanel.ResetPaint();
	}
````


## 幽淬相关的修改

**【原版代码】** 
```csharp
public bool SoulCrystalYouPowerUp(float badd = 0f, float irate = 0f, int v = 1)
{
	if (base.Rate >= 12)
	{
		return false;
	}
	if (World.RandomRate(Mathf.Pow(0.9f + badd, (float)(base.Rate + this.YouPower))))//幂函数，0.9为底数，次方数=品阶+幽淬次数
	{
		ItemThing itemThing;
		if (base.Count == 1)
		{
			itemThing = this;
		}
		else
		{
			itemThing = this.Split(1, true);
			base.map.DropItem(itemThing, base.Key, true, true, true, false, 0f);
			(UnityEngine.Object.Instantiate(Resources.Load("Effect/System/FlyLine")) as GameObject).GetComponent<FlyLineRender>().Begin(base.Pos, itemThing.Pos, 0.2f, null);
		}
		itemThing.YouPower += v; //幽淬次数+1
		itemThing.Rate += v;	 //品阶+1
		if (itemThing.View != null && itemThing.Rate >= 3)
		{
			itemThing.View.ShowItemRay(new Color?(GameDefine.GetRateColor(itemThing.Rate)));
			itemThing.NeedClick = true;
		}
		GameWatch.Instance.PlayUIAudio("Sound/ding");
		return true;
	}
	return false;
}
````

**【修改内容】** 
```csharp
public bool SoulCrystalYouPowerUp(float badd = 0f, float irate = 0f, int v = 1)
{
	if (base.Rate >= 12)//当品阶大于等于12
	{
		if (this.IsFaBao)//判断是否为法宝
		{
			this.Fabao.AddGodCount(1);//天劫洗练次数加一
			if (this.View != null && base.Rate >= 3)//如果品阶大于等于3,则添加发光，玩家需要点击取消发光
			{
				this.View.ShowItemRay(new Color?(GameDefine.GetRateColor(base.Rate)));
				this.NeedClick = true;
			}
			GameWatch.Instance.PlayUIAudio("Sound/ding");//播放“叮”的一声
			return true;
		}
		return false;
	}
	else
	{
		if (World.RandomRate(Mathf.Pow(1f + badd, (float)(base.Rate + this.YouPower))))//底数改为1则100%成功
		{
			int num = Mathf.Min(10, base.Count);//判断物品本身数量和10做比较，最多循环10次。
			for (int i = 0; i < num; i++)//根据上一步算出的数量循环
			{
				ItemThing itemThing;
				if (base.Count == 1)
				{
					itemThing = this;
				}
				else
				{
					itemThing = this.Split(1, true);
					base.map.DropItem(itemThing, base.Key, true, true, true, false, 0f);
					(UnityEngine.Object.Instantiate(Resources.Load("Effect/System/FlyLine")) as GameObject).GetComponent<FlyLineRender>().Begin(base.Pos, itemThing.Pos, 0.2f, null);
				}
				itemThing.YouPower = 12 - itemThing.Rate;//计算幽淬多少次才能满12阶，次数会影响丹药效果。
				itemThing.Rate = 12;//品阶改成12阶
				if (itemThing.View != null && itemThing.Rate >= 3)
				{
					itemThing.View.ShowItemRay(new Color?(GameDefine.GetRateColor(itemThing.Rate)));
					itemThing.NeedClick = true;
				}
			}
			GameWatch.Instance.PlayUIAudio("Sound/ding");
			return true;
		}
		return false;
	}
}
````


## 灵淬相关的修改


**【原版代码】**
```csharp
public bool SoulCrystalLingPowerUp(float badd = 0f)
{
	if (base.Accommodate <= 0f)
	{
		return false;
	}
	if (World.RandomRate(Mathf.Pow(0.9f + badd, (float)(base.Rate + this.YouPower))))//底数0.9，越淬概率越低
	{
		ItemThing itemThing;
		if (base.Count == 1)
		{
			itemThing = this;
		}
		else
		{
			itemThing = this.Split(1, true);
			base.map.DropItem(itemThing, base.Key, true, true, true, false, 0f);
			(UnityEngine.Object.Instantiate(Resources.Load("Effect/System/FlyLine")) as GameObject).GetComponent<FlyLineRender>().Begin(base.Pos, itemThing.Pos, 0.2f, null);
		}
		itemThing.LingPower++;//灵淬次数加一
		if (itemThing.IsFaBao)//判断是否为法宝
		{
			float property = itemThing.Fabao.GetProperty(g_emFaBaoP.MaxLing);//获取现有最大灵力
			itemThing.Fabao.SetProperty(g_emFaBaoP.MaxLing, property * 1.1f);//最大灵力*1.1，每次增加最大灵力10%
		}
		else
		{
			itemThing.AccommodateAddv += 5f;
		}
		GameWatch.Instance.PlayUIAudio("Sound/ding");
		return true;
	}
	return false;
}
```




**【修改内容】** 
```csharp
public bool SoulCrystalLingPowerUp(float badd = 0f)
{
	if (base.Accommodate <= 0f && !this.IsFaBao)//加上条件跳过法宝
	{
		return false;
	}
	if (World.RandomRate(Mathf.Pow(1f + badd, (float)(base.Rate + this.YouPower))))//底数改成1，概率100%
	{
		ItemThing itemThing;
		if (base.Count == 1)
		{
			itemThing = this;
		}
		else
		{
			itemThing = this.Split(1, true);
			base.map.DropItem(itemThing, base.Key, true, true, true, false, 0f);
			(UnityEngine.Object.Instantiate(Resources.Load("Effect/System/FlyLine")) as GameObject).GetComponent<FlyLineRender>().Begin(base.Pos, itemThing.Pos, 0.2f, null);
		}
		itemThing.LingPower++;//灵淬次数+1
		if (itemThing.IsFaBao)//判断是否为法宝
		{
			float property = itemThing.Fabao.GetProperty(g_emFaBaoP.MaxLing);//获取灵力最大值
			itemThing.Fabao.SetProperty(g_emFaBaoP.MaxLing, property * 1.1f);//灵力最大值增加10%
			float property2 = itemThing.Fabao.GetProperty(g_emFaBaoP.AttackPower);//获取威力
			itemThing.Fabao.SetProperty(g_emFaBaoP.AttackPower, property2 * 1.1f);//威力增加10%
			float property3 = itemThing.Fabao.GetProperty(g_emFaBaoP.RotSpeed);//获取转速
			itemThing.Fabao.SetProperty(g_emFaBaoP.RotSpeed, property3 * 1.1f);//转速增加10%
			float property4 = itemThing.Fabao.GetProperty(g_emFaBaoP.LingRecover);//获取灵力回复速度
			itemThing.Fabao.SetProperty(g_emFaBaoP.LingRecover, property4 * 1.1f);//灵力回复增加10%
			float property5 = itemThing.Fabao.GetProperty(g_emFaBaoP.Scale);//获取体积
			itemThing.Fabao.SetProperty(g_emFaBaoP.Scale, property5 * 1.1f);//体积增加10%
			float property6 = itemThing.Fabao.GetProperty(g_emFaBaoP.TailLenght);//获取拖尾长度
			itemThing.Fabao.SetProperty(g_emFaBaoP.TailLenght, property6 * 1.1f);//拖尾长度增加10%
			float property7 = itemThing.Fabao.GetProperty(g_emFaBaoP.KnockBackAddition);//获取击退能力
			itemThing.Fabao.SetProperty(g_emFaBaoP.KnockBackAddition, property7 * 1.1f);//击退能力增加10%
			float property8 = itemThing.Fabao.GetProperty(g_emFaBaoP.KnockBackResistance);//获取击退抵抗
			itemThing.Fabao.SetProperty(g_emFaBaoP.KnockBackResistance, property8 * 1.1f);//击退抵抗增加10%
			itemThing.Fabao.SetProperty(g_emFaBaoP.AttackRate, 0.3f);//攻击频率固定0.3/s
		}
		else
		{
			itemThing.AccommodateAddv += 5f;
		}
		GameWatch.Instance.PlayUIAudio("Sound/ding");
		return true;
	}
	return false;
}

```




[![Deploy CFN Template in us-west-2](./images/deploy-to-aws.png)](https://console.aws.amazon.com/cloudformation/home?region=us-west-2#/stacks/new?stackName=GuardDuty-Hands-On&templateURL=https://s3-us-west-2.amazonaws.com/sa-security-specialist-workshops-us-west-2/guardduty-hands-on/guardduty-cfn-template.yml)



Table CSS test

ID | Item | Remarks
---|------|--------
1  |灵淬  |测试
2| 幽淬| 测试

### Title

* [Getting Started](#started) 
* [Deploy the Environment](#deploy) 
* [Scenario 1 – Compromised EC2 Instance](#attack1) 
* [Scenario 2 – Compromised IAM Credentials](#attack2)
* [Scenario 3 – IAM Role Credential Exfiltration](#attack3)
* [Summary](#summary)
* [Clean Up](#cleanup)

**First Click**:

> Quote block

````
  Code block
````


#### Scenario 1 – Compromised EC2 instance <a name="attack1"/> </a>



After an uneventful yet unnecessarily long commute to work, you arrived at the office on Monday morning. You grabbed a cup of coffee, sat down in your cube, opened up your laptop and begin to go through your emails. Soon after you begin though you start receiving emails indicating that GuardDuty has detected new threats. You don’t yet know the extent of the threats but you quickly begin to investigate. Now the good news is that your coworker Alice has already set up some hooks for specific findings so that they will be automatically remediated. 


#### Questions
* Which data source did GuardDuty use to identify this threat?
* Will isolating the instance have any effect on an application running on the instance?
* How could you add more detail to the email notifications?

#### Deploy the environment <a name="deploy"/> </a>



S/N	|		Image	
----|		----	
1	|	![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000 (44%29.png)
2	|	![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000 (67%29.png)
3	|	![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000.png)
4	|	![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000 (48%29.png)
5	|	![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000 (54%29.png)
6	|	![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000 (67%29.png)
7	|	![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000 (68%29.png)
8	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000 (113%29.png)
9	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000 (67%29.png)
10	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000 (74%29.png)
11	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000 (76%29.png)
12	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000 (95%29.png)
13	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000.png)
14	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000 (31%29.png)
15	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000 (34%29.png)
16	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000 (41%29.png)
17	|	![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000 (48%29.png)
18	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shan1000 (2%29.png)
19	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shanzi1000 (2%29.png)
20	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shanzi1000.png)
21	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shipin1000 (3%29.png)
22	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shipin1000 (7%29.png)
23	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000 (21%29.png)
24	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000 (27%29.png)
25	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000 (29%29.png)
26	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000 (30%29.png)
27	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (1%29.png)
28	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (19%29.png)
29	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (24%29.png)
30	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (3%29.png)
31	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (38%29.png)
32	|	![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000 (50%29.png)
33	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (23%29.png)
34	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (28%29.png)
35	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (3%29.png)
36	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (33%29.png)
37	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (46%29.png)
38	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (48%29.png)
39	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (59%29.png)
40	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (70%29.png)
41	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000 (72%29.png)
42	|	![](./Mods/Tiancai/Resources/Spr/Building/baowuzhi.png)
43	|	![](./Mods/Tiancai/Resources/Spr/Building/building_jianlingtai01a.png)
44	|	![](./Mods/Tiancai/Resources/Spr/Building/card (320%29.png)
45	|	![](./Mods/Tiancai/Resources/Spr/Building/card (322%29.png)
46	|	![](./Mods/Tiancai/Resources/Spr/Building/fazuo.png)
47	|	![](./Mods/Tiancai/Resources/Spr/Building/huishou.png)
48	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen1.png)
49	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen2.png)
50	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen3.png)
51	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen4.png)
52	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen5.png)
53	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen6.png)
54	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen7.png)
55	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen8.png)
56	|	![](./Mods/Tiancai/Resources/Spr/Building/jianzhen9.png)
57	|	![](./Mods/Tiancai/Resources/Spr/Building/kuangshan.png)
58	|	![](./Mods/Tiancai/Resources/Spr/Building/ldk1001.png)
59	|	![](./Mods/Tiancai/Resources/Spr/Building/lingbaochui.png)
60	|	![](./Mods/Tiancai/Resources/Spr/Building/lingshifang.png)
61	|	![](./Mods/Tiancai/Resources/Spr/Building/tianyifang.png)
62	|	![](./Mods/Tiancai/Resources/Spr/Building/xiulianzhi.png)
63	|	![](./Mods/Tiancai/Resources/Spr/Building/xuanguangjian.png)
64	|	![](./Mods/Tiancai/Resources/Spr/Building/yaozushizhe.png)
65	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu1.png)
66	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu2.png)
67	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu3.png)
68	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu4.png)
69	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu5.png)
70	|	![](./Mods/Tiancai/Resources/Spr/Fabao/baowu6.png)
71	|	![](./Mods/Tiancai/Resources/Spr/Fabao/bian1.png)
72	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ci1.png)
73	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ci2.png)
74	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ci3.png)
75	|	![](./Mods/Tiancai/Resources/Spr/Fabao/dao1.png)
76	|	![](./Mods/Tiancai/Resources/Spr/Fabao/dao2.png)
77	|	![](./Mods/Tiancai/Resources/Spr/Fabao/dao4.png)
78	|	![](./Mods/Tiancai/Resources/Spr/Fabao/fu1.png)
79	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan1.png)
80	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan2.png)
81	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan3.png)
82	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan4.png)
83	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan5.png)
84	|	![](./Mods/Tiancai/Resources/Spr/Fabao/huan6.png)
85	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ji1.png)
86	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ji2.png)
87	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ji3.png)
88	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ji4.png)
89	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian1.png)
90	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian10.png)
91	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian11.png)
92	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian12.png)
93	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian13.png)
94	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian14.png)
95	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian2.png)
96	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian3.png)
97	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian4.png)
98	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian5.png)
99	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian6.png)
100	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian7.png)
101	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian8.png)
102	|	![](./Mods/Tiancai/Resources/Spr/Fabao/jian9.png)
103	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qi1.png)
104	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin1.png)
105	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin2.png)
106	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin3.png)
107	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin4.png)
108	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin5.png)
109	|	![](./Mods/Tiancai/Resources/Spr/Fabao/qin6.png)
110	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan1.png)
111	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan10.png)
112	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan11.png)
113	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan2.png)
114	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan3.png)
115	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan4.png)
116	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan5.png)
117	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan6.png)
118	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan7.png)
119	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan8.png)
120	|	![](./Mods/Tiancai/Resources/Spr/Fabao/shan9.png)
121	|	![](./Mods/Tiancai/Resources/Spr/Fabao/ta1.png)
122	|	![](./Mods/Tiancai/Resources/Spr/Fabao/wang1.png)
123	|	![](./Mods/Tiancai/Resources/Spr/Fabao/yi1.png)
124	|	![](./Mods/Tiancai/Resources/Spr/Fabao/yi2.png)
125	|	![](./Mods/Tiancai/Resources/Spr/Fabao/yin1.png)
126	|	![](./Mods/Tiancai/Resources/Spr/Fabao/zan1.png)
127	|	![](./Mods/Tiancai/Resources/Spr/Fabao/zan2.png)
128	|	![](./Mods/Tiancai/Resources/Spr/Fabao/zhu1.png)
129	|	![](./Mods/Tiancai/Resources/Spr/Fabao/zhuo1.png)
130	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/1lingshi.png)
131	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi1.png)
132	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi10.png)
133	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi11.png)
134	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi2.png)
135	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi3.png)
136	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi4.png)
137	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi5.png)
138	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi6.png)
139	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi7.png)
140	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi8.png)
141	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi9.png)
142	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi1.png)
143	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi10.png)
144	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi11.png)
145	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi2.png)
146	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi3.png)
147	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi4.png)
148	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi5.png)
149	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi6.png)
150	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi7.png)
151	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi8.png)
152	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi9.png)
153	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi1.png)
154	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi10.png)
155	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi11.png)
156	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi2.png)
157	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi3.png)
158	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi4.png)
159	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi5.png)
160	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi6.png)
161	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi7.png)
162	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi8.png)
163	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi9.png)
164	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi1.png)
165	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi10.png)
166	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi11.png)
167	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi2.png)
168	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi3.png)
169	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi4.png)
170	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi5.png)
171	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi6.png)
172	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi7.png)
173	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi8.png)
174	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi9.png)
175	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi1.png)
176	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi10.png)
177	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi11.png)
178	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi2.png)
179	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi3.png)
180	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi4.png)
181	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi5.png)
182	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi6.png)
183	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi7.png)
184	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi8.png)
185	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi9.png)
186	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi1.png)
187	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi10.png)
188	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi11.png)
189	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi2.png)
190	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi3.png)
191	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi4.png)
192	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi5.png)
193	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi6.png)
194	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi7.png)
195	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi8.png)
196	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi9.png)
197	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi1.png)
198	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi10.png)
199	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi11.png)
200	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi2.png)
201	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi3.png)
202	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi4.png)
203	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi5.png)
204	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi6.png)
205	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi7.png)
206	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi8.png)
207	|	![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi9.png)
208	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/1.png)
209	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/2.png)
210	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/3.png)
211	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/4.png)
212	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/5.png)
213	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/lingzhu.png)
214	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/s1.png)
215	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/s2.png)
216	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/s3.png)
217	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/s4.png)
218	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/s5.png)
219	|	![](./Mods/Tiancai/Resources/Spr/Lingzhu/z1.png)
220	|	![](./Mods/Tiancai/Resources/Spr/Object/1.png)
221	|	![](./Mods/Tiancai/Resources/Spr/Object/BIYE.png)
222	|	![](./Mods/Tiancai/Resources/Spr/Object/DMXF.png)
223	|	![](./Mods/Tiancai/Resources/Spr/Object/FHYJ.png)
224	|	![](./Mods/Tiancai/Resources/Spr/Object/HUO.png)
225	|	![](./Mods/Tiancai/Resources/Spr/Object/JIN.png)
226	|	![](./Mods/Tiancai/Resources/Spr/Object/MU.png)
227	|	![](./Mods/Tiancai/Resources/Spr/Object/NEIMEN.png)
228	|	![](./Mods/Tiancai/Resources/Spr/Object/QT1.png)
229	|	![](./Mods/Tiancai/Resources/Spr/Object/QT10.png)
230	|	![](./Mods/Tiancai/Resources/Spr/Object/QT11.png)
231	|	![](./Mods/Tiancai/Resources/Spr/Object/QT12.png)
232	|	![](./Mods/Tiancai/Resources/Spr/Object/QT13.png)
233	|	![](./Mods/Tiancai/Resources/Spr/Object/QT14.png)
234	|	![](./Mods/Tiancai/Resources/Spr/Object/QT15.png)
235	|	![](./Mods/Tiancai/Resources/Spr/Object/QT2.png)
236	|	![](./Mods/Tiancai/Resources/Spr/Object/QT3.png)
237	|	![](./Mods/Tiancai/Resources/Spr/Object/QT4.png)
238	|	![](./Mods/Tiancai/Resources/Spr/Object/QT5.png)
239	|	![](./Mods/Tiancai/Resources/Spr/Object/QT6.png)
240	|	![](./Mods/Tiancai/Resources/Spr/Object/QT7.png)
241	|	![](./Mods/Tiancai/Resources/Spr/Object/QT8.png)
242	|	![](./Mods/Tiancai/Resources/Spr/Object/QT9.png)
243	|	![](./Mods/Tiancai/Resources/Spr/Object/SHUI.png)
244	|	![](./Mods/Tiancai/Resources/Spr/Object/TGHMS.png)
245	|	![](./Mods/Tiancai/Resources/Spr/Object/TGXF.png)
246	|	![](./Mods/Tiancai/Resources/Spr/Object/TU.png)
247	|	![](./Mods/Tiancai/Resources/Spr/Object/TYSD.png)
248	|	![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_1.png)
249	|	![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_2.png)
250	|	![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_3.png)
251	|	![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_4.png)
252	|	![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_5.png)
253	|	![](./Mods/Tiancai/Resources/Spr/Object/WAIMEN.png)
254	|	![](./Mods/Tiancai/Resources/Spr/Object/XXFB.png)
255	|	![](./Mods/Tiancai/Resources/Spr/Object/XXHF.png)
256	|	![](./Mods/Tiancai/Resources/Spr/Object/XXLL.png)
257	|	![](./Mods/Tiancai/Resources/Spr/Object/XXLQ.png)
258	|	![](./Mods/Tiancai/Resources/Spr/Object/XXQL.png)
259	|	![](./Mods/Tiancai/Resources/Spr/Object/XXSF.png)
260	|	![](./Mods/Tiancai/Resources/Spr/Object/XXSZ.png)
261	|	![](./Mods/Tiancai/Resources/Spr/Object/XXTN.png)
262	|	![](./Mods/Tiancai/Resources/Spr/Object/XXXL.png)
263	|	![](./Mods/Tiancai/Resources/Spr/Object/YT1.png)
264	|	![](./Mods/Tiancai/Resources/Spr/Object/YT10.png)
265	|	![](./Mods/Tiancai/Resources/Spr/Object/YT11.png)
266	|	![](./Mods/Tiancai/Resources/Spr/Object/YT12.png)
267	|	![](./Mods/Tiancai/Resources/Spr/Object/YT13.png)
268	|	![](./Mods/Tiancai/Resources/Spr/Object/YT2.png)
269	|	![](./Mods/Tiancai/Resources/Spr/Object/YT3.png)
270	|	![](./Mods/Tiancai/Resources/Spr/Object/YT4.png)
271	|	![](./Mods/Tiancai/Resources/Spr/Object/YT5.png)
272	|	![](./Mods/Tiancai/Resources/Spr/Object/YT6.png)
273	|	![](./Mods/Tiancai/Resources/Spr/Object/YT7.png)
274	|	![](./Mods/Tiancai/Resources/Spr/Object/YT8.png)
275	|	![](./Mods/Tiancai/Resources/Spr/Object/YT9.png)
276	|	![](./Mods/Tiancai/Resources/Spr/Object/YTCJG.png)
277	|	![](./Mods/Tiancai/Resources/Spr/Object/YTXLT.png)
278	|	![](./Mods/Tiancai/Resources/Spr/Object/YTXY.png)
279	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua1.png)
280	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua2.png)
281	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua3.png)
282	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua4.png)
283	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua5.png)
284	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua6.png)
285	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua7.png)
286	|	![](./Mods/Tiancai/Resources/Spr/Object/bagua8.png)
287	|	![](./Mods/Tiancai/Resources/Spr/Object/bang.png)
288	|	![](./Mods/Tiancai/Resources/Spr/Object/car.png)
289	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao.png)
290	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao1.png)
291	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao2.png)
292	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao3.png)
293	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao4.png)
294	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao5.png)
295	|	![](./Mods/Tiancai/Resources/Spr/Object/danyao6.png)
296	|	![](./Mods/Tiancai/Resources/Spr/Object/gang.png)
297	|	![](./Mods/Tiancai/Resources/Spr/Object/gang2.png)
298	|	![](./Mods/Tiancai/Resources/Spr/Object/huolinggen.png)
299	|	![](./Mods/Tiancai/Resources/Spr/Object/jinlinggen.png)
300	|	![](./Mods/Tiancai/Resources/Spr/Object/jinqian1.png)
301	|	![](./Mods/Tiancai/Resources/Spr/Object/jinxiazi.png)
302	|	![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi1.png)
303	|	![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi2.png)
304	|	![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi3.png)
305	|	![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi4.png)
306	|	![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi5.png)
307	|	![](./Mods/Tiancai/Resources/Spr/Object/lanchouduan.png)
308	|	![](./Mods/Tiancai/Resources/Spr/Object/lanlingluo.png)
309	|	![](./Mods/Tiancai/Resources/Spr/Object/ldk1001.jpg	)
310	|	![](./Mods/Tiancai/Resources/Spr/Object/lingshidai1.png)
311	|	![](./Mods/Tiancai/Resources/Spr/Object/lingshidai2.png)
312	|	![](./Mods/Tiancai/Resources/Spr/Object/lingshixiang1.png)
313	|	![](./Mods/Tiancai/Resources/Spr/Object/lingshixiang2.png)
314	|	![](./Mods/Tiancai/Resources/Spr/Object/macbookair.png)
315	|	![](./Mods/Tiancai/Resources/Spr/Object/mulinggen.png)
316	|	![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao031.png)
317	|	![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao032.png)
318	|	![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao033.png)
319	|	![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao034.png)
320	|	![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao035.png)
321	|	![](./Mods/Tiancai/Resources/Spr/Object/object_lingshi01.png)
322	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi01.png)
323	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi02.png)
324	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi03.png)
325	|	![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi04.png)
326	|	![](./Mods/Tiancai/Resources/Spr/Object/object_stariron.png)
327	|	![](./Mods/Tiancai/Resources/Spr/Object/object_stariron2.png)
328	|	![](./Mods/Tiancai/Resources/Spr/Object/object_stariron3.png)
329	|	![](./Mods/Tiancai/Resources/Spr/Object/object_stariron4.png)
330	|	![](./Mods/Tiancai/Resources/Spr/Object/object_stariron5.png)
331	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi1.png)
332	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi2.png)
333	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi3.png)
334	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi4.png)
335	|	![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi5.png)
336	|	![](./Mods/Tiancai/Resources/Spr/Object/object_weapongong01.png)
337	|	![](./Mods/Tiancai/Resources/Spr/Object/object_weaponjian01.png)
338	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yusui01.png)
339	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yusui02.png)
340	|	![](./Mods/Tiancai/Resources/Spr/Object/object_yusui03.png)
341	|	![](./Mods/Tiancai/Resources/Spr/Object/pojiudemuxia.png)
342	|	![](./Mods/Tiancai/Resources/Spr/Object/qianpiao.png)
343	|	![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan.png)
344	|	![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan0.png)
345	|	![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan1.png)
346	|	![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan2.png)
347	|	![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan3.png)
348	|	![](./Mods/Tiancai/Resources/Spr/Object/shuilinggen.png)
349	|	![](./Mods/Tiancai/Resources/Spr/Object/smartphone.png)
350	|	![](./Mods/Tiancai/Resources/Spr/Object/suipian.png)
351	|	![](./Mods/Tiancai/Resources/Spr/Object/taiyiling.png)
352	|	![](./Mods/Tiancai/Resources/Spr/Object/tulinggen.png)
353	|	![](./Mods/Tiancai/Resources/Spr/Object/xiangzi1.png)
354	|	![](./Mods/Tiancai/Resources/Spr/Object/xiangzi2.png)
355	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian.png)
356	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian1.png)
357	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian2.png)
358	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian3.png)
359	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian4.png)
360	|	![](./Mods/Tiancai/Resources/Spr/Object/youmingjian5.png)
361	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi1.png)
362	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi2.png)
363	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi3.png)
364	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi4.png)
365	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi5.png)
366	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao1.png)
367	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao2.png)
368	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao3.png)
369	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao4.png)
370	|	![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao5.png)
371	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/1.png)
372	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/10.png)
373	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/11.png)
374	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/12.png)
375	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/13.png)
376	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/2.png)
377	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/3.png)
378	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/4.png)
379	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/5.png)
380	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/6.png)
381	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/7.png)
382	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/8.png)
383	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/9.png)
384	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/huo1.png)
385	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/huo2.png)
386	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/huo3.png)
387	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/jin1.png)
388	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/jin2.png)
389	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/jin3.png)
390	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/mu1.png)
391	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/mu2.png)
392	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/mu3.png)
393	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/renfu.png)
394	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/shui1.png)
395	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/shui2.png)
396	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/shui3.png)
397	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/tianfu.png)
398	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/tu1.png)
399	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/tu2.png)
400	|	![](./Mods/Tiancai/Resources/Spr/Tubiao/tu3.png)
401	|	![](./Mods/Tiancai/Resources/Spr/Ui/BQT.png)
402	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_1.png)
403	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_2.png)
404	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_3.png)
405	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_4.png)
406	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_5.png)
407	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_6.png)
408	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_7.png)
409	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_a1.png)
410	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_a2.png)
411	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_a3.png)
412	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_a4.png)
413	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_e1.png)
414	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k1.png)
415	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k2.png)
416	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k3.png)
417	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k4.png)
418	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k5.png)
419	|	![](./Mods/Tiancai/Resources/Spr/Ui/DM_k6.png)
420	|	![](./Mods/Tiancai/Resources/Spr/Ui/DTT.png)
421	|	![](./Mods/Tiancai/Resources/Spr/Ui/FHT.png)
422	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_1.png)
423	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_2.png)
424	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_3.png)
425	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_4.png)
426	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_5.png)
427	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_6.png)
428	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_7.png)
429	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_8.png)
430	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_9.png)
431	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_a1.png)
432	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_a2.png)
433	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b1.png)
434	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b2.png)
435	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b3.png)
436	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b4.png)
437	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b5.png)
438	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_b6.png)
439	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_c1.png)
440	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_c2.png)
441	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_c3.png)
442	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_c4.png)
443	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_c5.png)
444	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_d1.png)
445	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_d2.png)
446	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_d3.png)
447	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_e1.png)
448	|	![](./Mods/Tiancai/Resources/Spr/Ui/FH_e2.png)
449	|	![](./Mods/Tiancai/Resources/Spr/Ui/HMT.png)
450	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_1.png)
451	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_2.png)
452	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_3.png)
453	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_4.png)
454	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_5.png)
455	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_6.png)
456	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_7.png)
457	|	![](./Mods/Tiancai/Resources/Spr/Ui/SY_e2.png)
458	|	![](./Mods/Tiancai/Resources/Spr/Ui/TGT.png)
459	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_1.png)
460	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_10.png)
461	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_11.png)
462	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_12.png)
463	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_13.png)
464	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_14.png)
465	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_2.png)
466	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_3.png)
467	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_4.png)
468	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_5.png)
469	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_6.png)
470	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_7.png)
471	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_8.png)
472	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_9.png)
473	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a1.png)
474	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a2.png)
475	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a3.png)
476	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a4.png)
477	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a5.png)
478	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_a6.png)
479	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b1.png)
480	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b2.png)
481	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b3.png)
482	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b4.png)
483	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b5.png)
484	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b6.png)
485	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_b7.png)
486	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c1.png)
487	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c10.png)
488	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c2.png)
489	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c3.png)
490	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c4.png)
491	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c5.png)
492	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c6.png)
493	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c7.png)
494	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c8.png)
495	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_c9.png)
496	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d1.png)
497	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d2.png)
498	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d3.png)
499	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d4.png)
500	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d5.png)
501	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_d6.png)
502	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_e1.png)
503	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_e2.png)
504	|	![](./Mods/Tiancai/Resources/Spr/Ui/TG_e3.png)
505	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/asd.png)
506	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/chui1.png)
507	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/chui2.png)
508	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/dao1.png)
509	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/dao2.png)
510	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou.png)
511	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou1.png)
512	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou2.png)
513	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou3.png)
514	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou4.png)
515	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/gong1.png)
516	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/gong2.png)
517	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/huan1.png)
518	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/huan2.png)
519	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jian.png)
520	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jian1.png)
521	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jian2.png)
522	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jians1.png)
523	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jians2.png)
524	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jians3.png)
525	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jians4.png)
526	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/jians5.png)
527	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/qiang1.png)
528	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/qiang2.png)
529	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/shangqingdaopao.png)
530	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/shangqingdaopao2.png)
531	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/taijidaopao.png)
532	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/taijidaopao2.png)
533	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/taiqingdaopao.png)
534	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/taiqingdaopao2.png)
535	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/wuzidaofu.png)
536	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/xianshenbaolu.png)
537	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/yuqingdaopao.png)
538	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/yuqingdaopao2.png)
539	|	![](./Mods/Tiancai/Resources/Spr/Wuqi/351/255/224 (2%29.png)
540	|	![](./Mods/Tiancai/Resources/Spr/Ziti/dao.png)
541	|	![](./Mods/Tiancai/Resources/Spr/Ziti/jian.png)
542	|	![](./Mods/Tiancai/Resources/Spr/Ziti/mo.png)
543	|	![](./Mods/Tiancai/Resources/Spr/Ziti/sanjie.png)
544	|	![](./Mods/Tiancai/Resources/Spr/Ziti/sha.png)
545	|	![](./Mods/Tiancai/Resources/Spr/Ziti/zhan.png)

