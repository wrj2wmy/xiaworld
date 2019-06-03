# XiaWorld
了不起的修仙模拟器MOD教程

## 在开始之前

明白修仙模拟器源代码的树形结构能更快的找到所需要的类。
![](./Resources/xiaworld_tree_map.png)

了解dnspy的搜索功能能更快的精准定位，找到所需要的代码。
![](./Resources/dnspy_search_panel.png)



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


## 大凶房掉三种魄的修改

**【原版代码】**
```csharp
To be added
```

**【修改内容】**
```csharp
if (World.RandomRate(a))
{
	this.SetSpecialFlag(g_emNpcSpecailFlag.FLAG_DROPSOULCRYSTAL, 1);
	ItemThing itemThing = ItemRandomMachine.RandomItem("Item_SoulCrystalYou", null, 0, 12, -1f, 1);
	if (itemThing != null)
	{
		itemThing.Author = this.GetName();
		base.map.DropItem(itemThing, base.Key, true, true, true, true, 0f);
	}
}
if (World.RandomRate(a * 0.5f))
{
	ItemThing itemThing2 = ItemRandomMachine.RandomItem("Item_SoulCrystalLing", null, 0, 12, -1f, 1);
	if (itemThing2 != null)
	{
		itemThing2.Author = this.GetName();
		base.map.DropItem(itemThing2, base.Key, true, true, true, true, 0f);
	}
}
if (World.RandomRate(a * 0.25f))
{
	ItemThing itemThing3 = ItemRandomMachine.RandomItem("Item_SoulCrystalNing", null, 0, 12, -1f, 1);
	if (itemThing3 != null)
	{
		itemThing3.Author = this.GetName();
		base.map.DropItem(itemThing3, base.Key, true, true, true, true, 0f);
	}
}
```

## 历练相关的修改

### 瞬间历练 + 瞬间驻扎

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
public void AddNpcMapExplore(Npc npc, string place, bool isstay = false)
{
	PlacesMgr.MapExploreData mapExploreData = new PlacesMgr.MapExploreData
	{
		NpcID = npc.ID,
		Place = place
	};
	mapExploreData.Story = World.RandomRange(1f, 8f) * 600f;
	mapExploreData.NeedTime = 0f;//把所需时间改成0
	mapExploreData.IsStay = isstay;
	this.MapExplors.Add(mapExploreData);
	float lingCost = this.GetLingCost(place);
	npc.AddLing(-lingCost);
	GameWatch.Instance.Achievement.AddCount(g_emAchievementCountKind.Explore_Count, 1f);
	SchoolMgr.Instance.AddSchoolData(g_emSchoolData.Explore, 1);
	GameWatch.Instance.BuryingPoint(BuryingPointType.Explore, 1);
}


public override void Step(float dt)
{
	if (WorldMgr.Instance.IsLoading)
	{
		return;
	}
	this.lisremove.Clear();
	foreach (PlacesMgr.MapExploreData mapExploreData in this.MapExplors)
	{
		mapExploreData.StageP += dt;
		Npc npc = ThingMgr.Instance.FindThingByID(mapExploreData.NpcID) as Npc;
		int stage = mapExploreData.Stage;
		if (stage != 0)
		{
			if (stage != 1)
			{
				if (stage == 2)
				{
					if (mapExploreData.IsStay)
					{
						npc.RemoveSpecialFlag(g_emNpcSpecailFlag.FLAG_EXPLORESTAY);
					}
					ToilBase toilBase = (npc.JobEngine.CurJob == null) ? null : npc.JobEngine.CurJob.GetCurToil();
					if (toilBase != null)
					{
						toilBase.SetProgress((mapExploreData.StageP + mapExploreData.NeedTime + 0.01f) / (mapExploreData.NeedTime * 2f + 0.01f));//把原本120f的保底时间改成0.01f。防止分母为0的情况。
					}
					this.ChekStoryOnTheWay(mapExploreData, dt);
					if (mapExploreData.StageP >= mapExploreData.NeedTime)
					{
						this.LeaveNpcMapExplore(mapExploreData);
						this.lisremove.Add(mapExploreData);
					}
				}
			}
			else
			{
				ToilBase toilBase2 = (npc.JobEngine.CurJob == null) ? null : npc.JobEngine.CurJob.GetCurToil();
				if (mapExploreData.IsStay)
				{
					if (!npc.HasSpecialFlag(g_emNpcSpecailFlag.FLAG_EXPLORESTAY))
					{
						npc.AddSpecialFlag(g_emNpcSpecailFlag.FLAG_EXPLORESTAY, 1);
					}
					if (toilBase2 != null)
					{
						toilBase2.SetProgress(0f);
					}
					mapExploreData.Story = 0f;//这里改为0直接触发下面的事件
					if (mapExploreData.Story <= 0f)
					{
						this.TriggerStory(mapExploreData.Place, mapExploreData.NpcID, false);
						mapExploreData.Story = World.RandomRange(2f, 4f) * 600f;
					}
					if (mapExploreData.StageP >= 0.5f)//这里可根据具体一次历练的时间而调整这个数值。瞬间历练修改后，0.1f约等于3次历练。 这里改成0.5f 约等于15次。
					{
						mapExploreData.StageP = 0f;
						mapExploreData.Stage = 2;
						//这里去掉了两行debuff的代码，从而没有了驻扎的CD
						GameEventMgr.Instance.TriggerEvent(10032, npc, null, null, false, 0);
					}
				}
				else
				{
					if (toilBase2 != null)
					{
						toilBase2.SetProgress((mapExploreData.StageP + mapExploreData.NeedTime) / (mapExploreData.NeedTime * 2f + 0.01f));//把原本120f的保底时间改成0.01f。防止分母为0的情况。
					}
					if (mapExploreData.StageP >= 0.01f)
					{
						this.TriggerStory(mapExploreData.Place, mapExploreData.NpcID, false);
						mapExploreData.StageP = 0f;
						mapExploreData.Stage = 2;
					}
				}
			}
		}
		else
		{
			this.ChekStoryOnTheWay(mapExploreData, dt);
			ToilBase toilBase3 = (npc.JobEngine.CurJob == null) ? null : npc.JobEngine.CurJob.GetCurToil();
			if (toilBase3 != null)
			{
				toilBase3.SetProgress(mapExploreData.StageP / (mapExploreData.NeedTime * 2f + 0.01f));//把原本120f的保底时间改成0.01f。防止分母为0的情况。
			}
			if (mapExploreData.StageP >= mapExploreData.NeedTime)
			{
				mapExploreData.Stage = 1;
				mapExploreData.StageP = 0f;
				PlacesMgr.PlaceData placeData = this.GetPlaceData(mapExploreData.Place);
				PlaceDef placeDef = this.GetPlaceDef(mapExploreData.Place);
				if (placeDef.ExploreNum > 0)
				{
					placeData.ArrivedCount += (int)(1f + npc.GetProperty("ExperienceFindSpeedAddV"));
				}
				this.TryUnlock(placeData, placeDef, mapExploreData.NpcID);
				if (placeData.ArrivedCount == 0)
				{
					placeData.ArrivedCount = 1;
					if (placeDef.Links != null && placeDef.Links.Count > 0)
					{
						bool flag = false;
						foreach (string name in placeDef.Links)
						{
							if (this.IsLocked(name))
							{
								flag = true;
								break;
							}
						}
						if (flag)
						{
							Npc npc2 = ThingMgr.Instance.FindThingByID(mapExploreData.NpcID) as Npc;
							MapStoryMgr.Instance.TriggerStorySelection("sys_openplacelink", npc2, mapExploreData.Place, null, null);
						}
					}
				}
				if (mapExploreData.IsStay)
				{
					mapExploreData.Story = 120f;
				}
			}
		}
		if (npc.Rank == g_emNpcRank.Disciple && npc.PropertyMgr.Practice.StoryBroken > 0f)
		{
			npc.PropertyMgr.Practice.StoryBroken = 0f;//这里改成0直接触发下面的事件，瞬间突破瓶颈。
			if (npc.PropertyMgr.Practice.StoryBroken <= 0f)
			{
				MapStoryMgr.Instance.TriggerStorySelection(npc.PropertyMgr.Practice.CurNeck.Story, npc, null, null, null);
			}
		}
	}
	foreach (PlacesMgr.MapExploreData item in this.lisremove)
	{
		this.MapExplors.Remove(item);
	}
}
```

### 真仙可历练

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
if (!this.UIInfo.m_goexp.grayed)
{
	this.UIInfo.m_goexp.onClick.Add(delegate()
	{
		Wnd_SelectNpc.Instance.Select(delegate(List<int> ids)
		{
			if (ids != null && ids.Count > 0)
			{
				foreach (int id in ids)
				{
					Npc npc = ThingMgr.Instance.FindThingByID(id) as Npc;
					if (npc.PropertyMgr.Practice.TouchNeck && npc.PropertyMgr.Practice.CurNeck != null && npc.PropertyMgr.Practice.CurNeck.NeckCountdown > 0f && !npc.HasSpecialFlag(g_emNpcSpecailFlag.FLAG_PRACTICEDIE) && npc.PropertyMgr.Practice.CurNeck.Kind != g_emGongBottleNeckType.Thunder && npc.PropertyMgr.Practice.CurNeck.Kind != g_emGongBottleNeckType.God)//这里的条件判断阻止真仙出门历练，所以加上条件使得真仙不满足条件就不会被阻止出门。
					{
						Wnd_Message.Show(string.Format("{0}的的瓶颈即将松动，暂时不宜外出历练。", npc.GetName()), 1, null, true, "历练", 0, 0, string.Empty);
					}
					else
					{
						float num = (PlacesMgr.Instance.GetCoat(npc, name) * 2f + 120f) / 600f;
						if (num <= 3f)
						{
							npc.AddCommand("GoMapExplore", new object[]
							{
								name
							});
						}
						else
						{
							Wnd_Message.Show(string.Format("此去路途遥远({2:F2}天)，确定要派遣{0}前往{1}吗？", npc.GetName(), def.DisplayName, num), 2, delegate(string s)
							{
								if (s == "1")
								{
									npc.AddCommand("GoMapExplore", new object[]
									{
										name
									});
								}
							}, true, null, 0, 0, string.Empty);
						}
					}
				}
			}
		}, g_emNpcRank.Disciple, 1, 10, null, (Npc npc) => npc.CheckCommandSingle("GoMapExplore") == null, string.Format("前往{0}", def.DisplayName), delegate(Npc npc)
		{
			float num = (PlacesMgr.Instance.GetCoat(npc, name) * 2f + 120f) / 600f;
			string text = string.Format("耗时：{0:N2}天\n", num);
			if (num > 3f)
			{
				text += "[color=#B65704]耗时较长[/color]\n";
			}
			return text;
		}, null, false);
	});
}
```

## 炼宝相关的修改

### 成功率100%

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
private void MakeFaBao()
{
	g_emItemLable kind = (base.Job.CMD as CommandRefiningFabao).kind;
	ItemThing itemThing = ThingMgr.Instance.FindThingByID(this.itemID) as ItemThing;
	//移除了原本成功概率计算的逻辑
	if (World.RandomRate(1f))//把这里改成1f就达到了100%成功的效果
	{
		BuildingThing inBuilding = base.npc.InBuilding;
		float num = base.npc.PropertyMgr.Luck * 1f + (float)GameDefine.FengShuiRefining[(int)inBuilding.FengShui];
		float num2 = 1f + World.RandomRange(num * -5f, num * 10f) / 100f;
		float lingfix = base.npc.GetProperty("FabaoMake_LingInheritRateAddV") * GameDefine.MindState2RefiningRate[base.npc.Needs.GetNeedLevel(g_emNeedType.MindState)] * num2;
		float qualityadd = base.npc.GetProperty("FabaoMake_QualityAddV") + base.npc.GetProperty("MadeQualityAddValue") + base.npc.GetProperty("ManufactureExpectQuality") * World.RandomRange(base.npc.GetProperty("ManufactureRealQualityPercentMin"), base.npc.GetProperty("ManufactureRealQualityPercentMax"));
		float godrate = base.npc.GetProperty("FabaoMake_TwelveRateChance") * base.npc.Needs.GetNeedValue(g_emNeedType.MindState) / 50f;
		ItemThing itemThing2 = ThingMgr.Instance.AddFaBao(itemThing, kind, lingfix, qualityadd, godrate);
		itemThing2.SetName(ThingMgr.Instance.RandomFabaoName(itemThing, new g_emPrestigeRank?(SchoolMgr.Instance.PrestigeRank)));
		itemThing2.NeedClick = true;
		itemThing2.Author = base.npc.GetName();
		base.npc.map.DropItem(itemThing2, base.npc.Key, true, true, false, false, 0f);
		if (MessageMgr.UseOldMessage)
		{
			MsgMgr.Instance.AddMsg(3, itemThing2, base.npc.GetName(), 0, true, false);
		}
		else
		{
			MessageMgr.Instance.AddMessage(38, new List<Thing>
			{
				itemThing2
			}, base.npc.GetName(), 0, 0, 0);
		}
		SchoolMgr.Instance.AddSchoolData(g_emSchoolData.Fabao, 1);
		if (itemThing2.Rate >= 12)
		{
			GameWatch.Instance.Achievement.UnLockAchievement(2001);
		}
		if (itemThing2.GetQuality() == 1f)
		{
			GameWatch.Instance.Achievement.UnLockAchievement(2000);
		}
		base.npc.PropertyMgr.Practice.AddTreeExp((float)(itemThing2.Rate * itemThing2.Rate) * 10f, false);
		if (base.npc.InBuilding.View != null)
		{
			base.npc.InBuilding.View.OnWorkFinished(true);
		}
		GameWatch.Instance.BuryingPoint(BuryingPointType.Refining, new Dictionary<string, object>
		{
			{
				"Stuff",
				itemThing.GetName()
			},
			{
				"Rate",
				itemThing2.Rate
			}
		});
		NpcMgr.Instance.TriggerNpcStory("RefiningSucceed", base.npc, null, itemThing2.GetName(), false, false);
	}
	else
	{
		base.npc.PropertyMgr.Practice.AddTreeExp(30f, false);
		NpcMgr.Instance.TriggerNpcStory("RefiningFailed", base.npc, null, itemThing.GetName(), false, false);
		if (MessageMgr.UseOldMessage)
		{
			MsgMgr.Instance.AddMsg(2, base.npc, null, 0, true, false);
		}
		else
		{
			MessageMgr.Instance.AddMessage(38, new List<Thing>
			{
				base.npc
			}, null, 1, 0, 0);
		}
		if (base.npc.InBuilding.View != null)
		{
			base.npc.InBuilding.View.OnWorkFinished(false);
		}
		GameWatch.Instance.BuryingPoint(BuryingPointType.Refining, 0);
	}
	GameWatch.Instance.Achievement.AddCount(g_emAchievementCountKind.Refining_Count, 1f);
	itemThing.SubCount(1);
}
```

### 神器概率100%

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
public void InitFabao(ItemThing oitem, g_emItemLable kind, float lingfix = 1f, float Qualityadd = 0f, float godrate = 1f)
{
	float x = oitem.LingV * lingfix;
	int num = oitem.Rate + (int)GameDefine.GetValueByMap<float>(GameDefine.Ling2FabaoRate, x);
	if (num < 12 || !World.RandomRate(godrate))//这一段没改，但是编译器自动修改了代码。
	{										   //这一段没改，但是编译器自动修改了代码。
	}										   //这一段没改，但是编译器自动修改了代码。
	num = 12;//直接强制品阶为12，达到100%获得神器的效果。
	this.InitFabao(num, oitem.def, (oitem.StuffDef == null) ? null : oitem.StuffDef.Name, oitem.GetName(), kind, Qualityadd, oitem.GetEquptMod(), oitem.ElementKind);
	this.Fabao.OName = oitem.GetName();
}
```

## 传功相关的修改

### 无视师徒关系传功

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
```

### 传功所需参悟减少

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
```

### 传功时间减少

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
```

## 门派人数上限的修改

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
//外门
public static int[] SchoolMaxNpc = new int[]
{
	24,
	24,
	36,
	48
};
//内门
public static int[] SchoolMaxDNpc = new int[]
{
	24,
	24,
	36,
	48
};
```

## 傀儡相关的修改

### 去除一人一傀儡的限制

**【原版代码】**
```csharp
```

**【修改内容】**
```csharp
public int CheckSpecialFlag(int name)
{
	int result = 0;
	this.SpecialFlag.TryGetValue(name, out result);
	//过滤绑定傀儡的两个标记达到一人多傀儡的效果。
	if ((g_emNpcSpecailFlag)name == g_emNpcSpecailFlag.NpcBindPuppet || (g_emNpcSpecailFlag)name == g_emNpcSpecailFlag.PuppetBindNpc)
		result = 0;
	return result;
}
```














## 图片资源


S/N	|	Category	|	Image	|	File
----	|	----	|	----	|	----
1	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000%20%2844%29.png)	|	object_kuilei1000%20%2844%29.png
2	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000%20%2867%29.png)	|	object_kuilei1000%20%2867%29.png
3	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_kuilei1000.png)	|	object_kuilei1000.png
4	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000%20%2848%29.png)	|	object_motian1000%20%2848%29.png
5	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000%20%2854%29.png)	|	object_motian1000%20%2854%29.png
6	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000%20%2867%29.png)	|	object_motian1000%20%2867%29.png
7	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_motian1000%20%2868%29.png)	|	object_motian1000%20%2868%29.png
8	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000%20%28113%29.png)	|	object_qita1000%20%28113%29.png
9	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000%20%2867%29.png)	|	object_qita1000%20%2867%29.png
10	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000%20%2874%29.png)	|	object_qita1000%20%2874%29.png
11	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000%20%2876%29.png)	|	object_qita1000%20%2876%29.png
12	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000%20%2895%29.png)	|	object_qita1000%20%2895%29.png
13	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qita1000.png)	|	object_qita1000.png
14	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000%20%2831%29.png)	|	object_qqsh1000%20%2831%29.png
15	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000%20%2834%29.png)	|	object_qqsh1000%20%2834%29.png
16	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000%20%2841%29.png)	|	object_qqsh1000%20%2841%29.png
17	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_qqsh1000%20%2848%29.png)	|	object_qqsh1000%20%2848%29.png
18	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shan1000%20%282%29.png)	|	object_shan1000%20%282%29.png
19	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shanzi1000%20%282%29.png)	|	object_shanzi1000%20%282%29.png
20	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shanzi1000.png)	|	object_shanzi1000.png
21	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shipin1000%20%283%29.png)	|	object_shipin1000%20%283%29.png
22	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shipin1000%20%287%29.png)	|	object_shipin1000%20%287%29.png
23	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000%20%2821%29.png)	|	object_tdyj1000%20%2821%29.png
24	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000%20%2827%29.png)	|	object_tdyj1000%20%2827%29.png
25	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000%20%2829%29.png)	|	object_tdyj1000%20%2829%29.png
26	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tdyj1000%20%2830%29.png)	|	object_tdyj1000%20%2830%29.png
27	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%281%29.png)	|	object_wuqi1000%20%281%29.png
28	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%2819%29.png)	|	object_wuqi1000%20%2819%29.png
29	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%2824%29.png)	|	object_wuqi1000%20%2824%29.png
30	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%283%29.png)	|	object_wuqi1000%20%283%29.png
31	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%2838%29.png)	|	object_wuqi1000%20%2838%29.png
32	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_wuqi1000%20%2850%29.png)	|	object_wuqi1000%20%2850%29.png
33	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2823%29.png)	|	object_yishou1000%20%2823%29.png
34	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2828%29.png)	|	object_yishou1000%20%2828%29.png
35	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%283%29.png)	|	object_yishou1000%20%283%29.png
36	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2833%29.png)	|	object_yishou1000%20%2833%29.png
37	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2846%29.png)	|	object_yishou1000%20%2846%29.png
38	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2848%29.png)	|	object_yishou1000%20%2848%29.png
39	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2859%29.png)	|	object_yishou1000%20%2859%29.png
40	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2870%29.png)	|	object_yishou1000%20%2870%29.png
41	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yishou1000%20%2872%29.png)	|	object_yishou1000%20%2872%29.png
42	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/baowuzhi.png)	|	baowuzhi.png
43	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/building_jianlingtai01a.png)	|	building_jianlingtai01a.png
44	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/card%20%28320%29.png)	|	card%20%28320%29.png
45	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/card%20%28322%29.png)	|	card%20%28322%29.png
46	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/fazuo.png)	|	fazuo.png
47	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/huishou.png)	|	huishou.png
48	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen1.png)	|	jianzhen1.png
49	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen2.png)	|	jianzhen2.png
50	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen3.png)	|	jianzhen3.png
51	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen4.png)	|	jianzhen4.png
52	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen5.png)	|	jianzhen5.png
53	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen6.png)	|	jianzhen6.png
54	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen7.png)	|	jianzhen7.png
55	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen8.png)	|	jianzhen8.png
56	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/jianzhen9.png)	|	jianzhen9.png
57	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/kuangshan.png)	|	kuangshan.png
58	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/ldk1001.png)	|	ldk1001.png
59	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/lingbaochui.png)	|	lingbaochui.png
60	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/lingshifang.png)	|	lingshifang.png
61	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/tianyifang.png)	|	tianyifang.png
62	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/xiulianzhi.png)	|	xiulianzhi.png
63	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/xuanguangjian.png)	|	xuanguangjian.png
64	|	Building	|	 ![](./Mods/Tiancai/Resources/Spr/Building/yaozushizhe.png)	|	yaozushizhe.png
65	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu1.png)	|	baowu1.png
66	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu2.png)	|	baowu2.png
67	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu3.png)	|	baowu3.png
68	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu4.png)	|	baowu4.png
69	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu5.png)	|	baowu5.png
70	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/baowu6.png)	|	baowu6.png
71	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/bian1.png)	|	bian1.png
72	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ci1.png)	|	ci1.png
73	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ci2.png)	|	ci2.png
74	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ci3.png)	|	ci3.png
75	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/dao1.png)	|	dao1.png
76	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/dao2.png)	|	dao2.png
77	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/dao4.png)	|	dao4.png
78	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/fu1.png)	|	fu1.png
79	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan1.png)	|	huan1.png
80	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan2.png)	|	huan2.png
81	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan3.png)	|	huan3.png
82	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan4.png)	|	huan4.png
83	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan5.png)	|	huan5.png
84	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/huan6.png)	|	huan6.png
85	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ji1.png)	|	ji1.png
86	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ji2.png)	|	ji2.png
87	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ji3.png)	|	ji3.png
88	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ji4.png)	|	ji4.png
89	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian1.png)	|	jian1.png
90	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian10.png)	|	jian10.png
91	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian11.png)	|	jian11.png
92	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian12.png)	|	jian12.png
93	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian13.png)	|	jian13.png
94	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian14.png)	|	jian14.png
95	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian2.png)	|	jian2.png
96	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian3.png)	|	jian3.png
97	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian4.png)	|	jian4.png
98	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian5.png)	|	jian5.png
99	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian6.png)	|	jian6.png
100	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian7.png)	|	jian7.png
101	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian8.png)	|	jian8.png
102	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/jian9.png)	|	jian9.png
103	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qi1.png)	|	qi1.png
104	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin1.png)	|	qin1.png
105	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin2.png)	|	qin2.png
106	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin3.png)	|	qin3.png
107	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin4.png)	|	qin4.png
108	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin5.png)	|	qin5.png
109	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/qin6.png)	|	qin6.png
110	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan1.png)	|	shan1.png
111	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan10.png)	|	shan10.png
112	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan11.png)	|	shan11.png
113	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan2.png)	|	shan2.png
114	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan3.png)	|	shan3.png
115	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan4.png)	|	shan4.png
116	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan5.png)	|	shan5.png
117	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan6.png)	|	shan6.png
118	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan7.png)	|	shan7.png
119	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan8.png)	|	shan8.png
120	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/shan9.png)	|	shan9.png
121	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/ta1.png)	|	ta1.png
122	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/wang1.png)	|	wang1.png
123	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/yi1.png)	|	yi1.png
124	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/yi2.png)	|	yi2.png
125	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/yin1.png)	|	yin1.png
126	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/zan1.png)	|	zan1.png
127	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/zan2.png)	|	zan2.png
128	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/zhu1.png)	|	zhu1.png
129	|	Fabao	|	 ![](./Mods/Tiancai/Resources/Spr/Fabao/zhuo1.png)	|	zhuo1.png
130	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/1lingshi.png)	|	1lingshi.png
131	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi1.png)	|	bailingshi1.png
132	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi10.png)	|	bailingshi10.png
133	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi11.png)	|	bailingshi11.png
134	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi2.png)	|	bailingshi2.png
135	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi3.png)	|	bailingshi3.png
136	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi4.png)	|	bailingshi4.png
137	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi5.png)	|	bailingshi5.png
138	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi6.png)	|	bailingshi6.png
139	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi7.png)	|	bailingshi7.png
140	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi8.png)	|	bailingshi8.png
141	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/bailingshi9.png)	|	bailingshi9.png
142	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi1.png)	|	heilingshi1.png
143	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi10.png)	|	heilingshi10.png
144	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi11.png)	|	heilingshi11.png
145	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi2.png)	|	heilingshi2.png
146	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi3.png)	|	heilingshi3.png
147	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi4.png)	|	heilingshi4.png
148	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi5.png)	|	heilingshi5.png
149	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi6.png)	|	heilingshi6.png
150	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi7.png)	|	heilingshi7.png
151	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi8.png)	|	heilingshi8.png
152	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/heilingshi9.png)	|	heilingshi9.png
153	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi1.png)	|	huolingshi1.png
154	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi10.png)	|	huolingshi10.png
155	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi11.png)	|	huolingshi11.png
156	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi2.png)	|	huolingshi2.png
157	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi3.png)	|	huolingshi3.png
158	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi4.png)	|	huolingshi4.png
159	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi5.png)	|	huolingshi5.png
160	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi6.png)	|	huolingshi6.png
161	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi7.png)	|	huolingshi7.png
162	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi8.png)	|	huolingshi8.png
163	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/huolingshi9.png)	|	huolingshi9.png
164	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi1.png)	|	jinlingshi1.png
165	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi10.png)	|	jinlingshi10.png
166	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi11.png)	|	jinlingshi11.png
167	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi2.png)	|	jinlingshi2.png
168	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi3.png)	|	jinlingshi3.png
169	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi4.png)	|	jinlingshi4.png
170	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi5.png)	|	jinlingshi5.png
171	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi6.png)	|	jinlingshi6.png
172	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi7.png)	|	jinlingshi7.png
173	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi8.png)	|	jinlingshi8.png
174	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/jinlingshi9.png)	|	jinlingshi9.png
175	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi1.png)	|	mulingshi1.png
176	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi10.png)	|	mulingshi10.png
177	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi11.png)	|	mulingshi11.png
178	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi2.png)	|	mulingshi2.png
179	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi3.png)	|	mulingshi3.png
180	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi4.png)	|	mulingshi4.png
181	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi5.png)	|	mulingshi5.png
182	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi6.png)	|	mulingshi6.png
183	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi7.png)	|	mulingshi7.png
184	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi8.png)	|	mulingshi8.png
185	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/mulingshi9.png)	|	mulingshi9.png
186	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi1.png)	|	shuilingshi1.png
187	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi10.png)	|	shuilingshi10.png
188	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi11.png)	|	shuilingshi11.png
189	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi2.png)	|	shuilingshi2.png
190	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi3.png)	|	shuilingshi3.png
191	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi4.png)	|	shuilingshi4.png
192	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi5.png)	|	shuilingshi5.png
193	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi6.png)	|	shuilingshi6.png
194	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi7.png)	|	shuilingshi7.png
195	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi8.png)	|	shuilingshi8.png
196	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/shuilingshi9.png)	|	shuilingshi9.png
197	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi1.png)	|	tulingshi1.png
198	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi10.png)	|	tulingshi10.png
199	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi11.png)	|	tulingshi11.png
200	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi2.png)	|	tulingshi2.png
201	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi3.png)	|	tulingshi3.png
202	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi4.png)	|	tulingshi4.png
203	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi5.png)	|	tulingshi5.png
204	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi6.png)	|	tulingshi6.png
205	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi7.png)	|	tulingshi7.png
206	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi8.png)	|	tulingshi8.png
207	|	Lingshi	|	 ![](./Mods/Tiancai/Resources/Spr/Lingshi/tulingshi9.png)	|	tulingshi9.png
208	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/1.png)	|	1.png
209	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/2.png)	|	2.png
210	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/3.png)	|	3.png
211	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/4.png)	|	4.png
212	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/5.png)	|	5.png
213	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/lingzhu.png)	|	lingzhu.png
214	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/s1.png)	|	s1.png
215	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/s2.png)	|	s2.png
216	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/s3.png)	|	s3.png
217	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/s4.png)	|	s4.png
218	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/s5.png)	|	s5.png
219	|	Lingzhu	|	 ![](./Mods/Tiancai/Resources/Spr/Lingzhu/z1.png)	|	z1.png
220	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/1.png)	|	1.png
221	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/BIYE.png)	|	BIYE.png
222	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/DMXF.png)	|	DMXF.png
223	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/FHYJ.png)	|	FHYJ.png
224	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/HUO.png)	|	HUO.png
225	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/JIN.png)	|	JIN.png
226	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/MU.png)	|	MU.png
227	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/NEIMEN.png)	|	NEIMEN.png
228	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT1.png)	|	QT1.png
229	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT10.png)	|	QT10.png
230	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT11.png)	|	QT11.png
231	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT12.png)	|	QT12.png
232	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT13.png)	|	QT13.png
233	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT14.png)	|	QT14.png
234	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT15.png)	|	QT15.png
235	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT2.png)	|	QT2.png
236	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT3.png)	|	QT3.png
237	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT4.png)	|	QT4.png
238	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT5.png)	|	QT5.png
239	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT6.png)	|	QT6.png
240	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT7.png)	|	QT7.png
241	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT8.png)	|	QT8.png
242	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/QT9.png)	|	QT9.png
243	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/SHUI.png)	|	SHUI.png
244	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TGHMS.png)	|	TGHMS.png
245	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TGXF.png)	|	TGXF.png
246	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TU.png)	|	TU.png
247	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYSD.png)	|	TYSD.png
248	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_1.png)	|	TYWJ_1.png
249	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_2.png)	|	TYWJ_2.png
250	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_3.png)	|	TYWJ_3.png
251	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_4.png)	|	TYWJ_4.png
252	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/TYWJ_5.png)	|	TYWJ_5.png
253	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/WAIMEN.png)	|	WAIMEN.png
254	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXFB.png)	|	XXFB.png
255	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXHF.png)	|	XXHF.png
256	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXLL.png)	|	XXLL.png
257	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXLQ.png)	|	XXLQ.png
258	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXQL.png)	|	XXQL.png
259	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXSF.png)	|	XXSF.png
260	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXSZ.png)	|	XXSZ.png
261	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXTN.png)	|	XXTN.png
262	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/XXXL.png)	|	XXXL.png
263	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT1.png)	|	YT1.png
264	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT10.png)	|	YT10.png
265	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT11.png)	|	YT11.png
266	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT12.png)	|	YT12.png
267	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT13.png)	|	YT13.png
268	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT2.png)	|	YT2.png
269	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT3.png)	|	YT3.png
270	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT4.png)	|	YT4.png
271	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT5.png)	|	YT5.png
272	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT6.png)	|	YT6.png
273	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT7.png)	|	YT7.png
274	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT8.png)	|	YT8.png
275	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YT9.png)	|	YT9.png
276	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YTCJG.png)	|	YTCJG.png
277	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YTXLT.png)	|	YTXLT.png
278	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/YTXY.png)	|	YTXY.png
279	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua1.png)	|	bagua1.png
280	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua2.png)	|	bagua2.png
281	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua3.png)	|	bagua3.png
282	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua4.png)	|	bagua4.png
283	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua5.png)	|	bagua5.png
284	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua6.png)	|	bagua6.png
285	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua7.png)	|	bagua7.png
286	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bagua8.png)	|	bagua8.png
287	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/bang.png)	|	bang.png
288	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/car.png)	|	car.png
289	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao.png)	|	danyao.png
290	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao1.png)	|	danyao1.png
291	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao2.png)	|	danyao2.png
292	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao3.png)	|	danyao3.png
293	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao4.png)	|	danyao4.png
294	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao5.png)	|	danyao5.png
295	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/danyao6.png)	|	danyao6.png
296	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/gang.png)	|	gang.png
297	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/gang2.png)	|	gang2.png
298	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/huolinggen.png)	|	huolinggen.png
299	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/jinlinggen.png)	|	jinlinggen.png
300	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/jinqian1.png)	|	jinqian1.png
301	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/jinxiazi.png)	|	jinxiazi.png
302	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi1.png)	|	kuangwuyuanshi1.png
303	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi2.png)	|	kuangwuyuanshi2.png
304	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi3.png)	|	kuangwuyuanshi3.png
305	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi4.png)	|	kuangwuyuanshi4.png
306	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/kuangwuyuanshi5.png)	|	kuangwuyuanshi5.png
307	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lanchouduan.png)	|	lanchouduan.png
308	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lanlingluo.png)	|	lanlingluo.png
309	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/ldk1001.jpg)	|	ldk1001.jpg
310	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lingshidai1.png)	|	lingshidai1.png
311	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lingshidai2.png)	|	lingshidai2.png
312	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lingshixiang1.png)	|	lingshixiang1.png
313	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/lingshixiang2.png)	|	lingshixiang2.png
314	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/macbookair.png)	|	macbookair.png
315	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/mulinggen.png)	|	mulinggen.png
316	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao031.png)	|	object_PINGdanyao031.png
317	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao032.png)	|	object_PINGdanyao032.png
318	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao033.png)	|	object_PINGdanyao033.png
319	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao034.png)	|	object_PINGdanyao034.png
320	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_PINGdanyao035.png)	|	object_PINGdanyao035.png
321	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_lingshi01.png)	|	object_lingshi01.png
322	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi01.png)	|	object_shixiazi01.png
323	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi02.png)	|	object_shixiazi02.png
324	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi03.png)	|	object_shixiazi03.png
325	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_shixiazi04.png)	|	object_shixiazi04.png
326	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_stariron.png)	|	object_stariron.png
327	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_stariron2.png)	|	object_stariron2.png
328	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_stariron3.png)	|	object_stariron3.png
329	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_stariron4.png)	|	object_stariron4.png
330	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_stariron5.png)	|	object_stariron5.png
331	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi1.png)	|	object_tianjiezhixi1.png
332	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi2.png)	|	object_tianjiezhixi2.png
333	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi3.png)	|	object_tianjiezhixi3.png
334	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi4.png)	|	object_tianjiezhixi4.png
335	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_tianjiezhixi5.png)	|	object_tianjiezhixi5.png
336	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_weapongong01.png)	|	object_weapongong01.png
337	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_weaponjian01.png)	|	object_weaponjian01.png
338	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yusui01.png)	|	object_yusui01.png
339	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yusui02.png)	|	object_yusui02.png
340	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/object_yusui03.png)	|	object_yusui03.png
341	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/pojiudemuxia.png)	|	pojiudemuxia.png
342	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qianpiao.png)	|	qianpiao.png
343	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan.png)	|	qingyunshan.png
344	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan0.png)	|	qingyunshan0.png
345	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan1.png)	|	qingyunshan1.png
346	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan2.png)	|	qingyunshan2.png
347	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/qingyunshan3.png)	|	qingyunshan3.png
348	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/shuilinggen.png)	|	shuilinggen.png
349	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/smartphone.png)	|	smartphone.png
350	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/suipian.png)	|	suipian.png
351	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/taiyiling.png)	|	taiyiling.png
352	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/tulinggen.png)	|	tulinggen.png
353	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/xiangzi1.png)	|	xiangzi1.png
354	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/xiangzi2.png)	|	xiangzi2.png
355	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian.png)	|	youmingjian.png
356	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian1.png)	|	youmingjian1.png
357	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian2.png)	|	youmingjian2.png
358	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian3.png)	|	youmingjian3.png
359	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian4.png)	|	youmingjian4.png
360	|	Object	|	 ![](./Mods/Tiancai/Resources/Spr/Object/youmingjian5.png)	|	youmingjian5.png
361	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi1.png)	|	shoupi1.png
362	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi2.png)	|	shoupi2.png
363	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi3.png)	|	shoupi3.png
364	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi4.png)	|	shoupi4.png
365	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/shoupi5.png)	|	shoupi5.png
366	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao1.png)	|	yumao1.png
367	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao2.png)	|	yumao2.png
368	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao3.png)	|	yumao3.png
369	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao4.png)	|	yumao4.png
370	|	Shoupi	|	 ![](./Mods/Tiancai/Resources/Spr/Shoupi/yumao5.png)	|	yumao5.png
371	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/1.png)	|	1.png
372	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/10.png)	|	10.png
373	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/11.png)	|	11.png
374	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/12.png)	|	12.png
375	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/13.png)	|	13.png
376	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/2.png)	|	2.png
377	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/3.png)	|	3.png
378	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/4.png)	|	4.png
379	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/5.png)	|	5.png
380	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/6.png)	|	6.png
381	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/7.png)	|	7.png
382	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/8.png)	|	8.png
383	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/9.png)	|	9.png
384	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/huo1.png)	|	huo1.png
385	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/huo2.png)	|	huo2.png
386	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/huo3.png)	|	huo3.png
387	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/jin1.png)	|	jin1.png
388	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/jin2.png)	|	jin2.png
389	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/jin3.png)	|	jin3.png
390	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/mu1.png)	|	mu1.png
391	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/mu2.png)	|	mu2.png
392	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/mu3.png)	|	mu3.png
393	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/renfu.png)	|	renfu.png
394	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/shui1.png)	|	shui1.png
395	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/shui2.png)	|	shui2.png
396	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/shui3.png)	|	shui3.png
397	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/tianfu.png)	|	tianfu.png
398	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/tu1.png)	|	tu1.png
399	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/tu2.png)	|	tu2.png
400	|	Tubiao	|	 ![](./Mods/Tiancai/Resources/Spr/Tubiao/tu3.png)	|	tu3.png
401	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/BQT.png)	|	BQT.png
402	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_1.png)	|	DM_1.png
403	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_2.png)	|	DM_2.png
404	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_3.png)	|	DM_3.png
405	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_4.png)	|	DM_4.png
406	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_5.png)	|	DM_5.png
407	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_6.png)	|	DM_6.png
408	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_7.png)	|	DM_7.png
409	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_a1.png)	|	DM_a1.png
410	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_a2.png)	|	DM_a2.png
411	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_a3.png)	|	DM_a3.png
412	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_a4.png)	|	DM_a4.png
413	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_e1.png)	|	DM_e1.png
414	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k1.png)	|	DM_k1.png
415	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k2.png)	|	DM_k2.png
416	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k3.png)	|	DM_k3.png
417	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k4.png)	|	DM_k4.png
418	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k5.png)	|	DM_k5.png
419	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DM_k6.png)	|	DM_k6.png
420	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/DTT.png)	|	DTT.png
421	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FHT.png)	|	FHT.png
422	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_1.png)	|	FH_1.png
423	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_2.png)	|	FH_2.png
424	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_3.png)	|	FH_3.png
425	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_4.png)	|	FH_4.png
426	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_5.png)	|	FH_5.png
427	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_6.png)	|	FH_6.png
428	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_7.png)	|	FH_7.png
429	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_8.png)	|	FH_8.png
430	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_9.png)	|	FH_9.png
431	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_a1.png)	|	FH_a1.png
432	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_a2.png)	|	FH_a2.png
433	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b1.png)	|	FH_b1.png
434	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b2.png)	|	FH_b2.png
435	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b3.png)	|	FH_b3.png
436	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b4.png)	|	FH_b4.png
437	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b5.png)	|	FH_b5.png
438	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_b6.png)	|	FH_b6.png
439	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_c1.png)	|	FH_c1.png
440	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_c2.png)	|	FH_c2.png
441	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_c3.png)	|	FH_c3.png
442	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_c4.png)	|	FH_c4.png
443	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_c5.png)	|	FH_c5.png
444	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_d1.png)	|	FH_d1.png
445	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_d2.png)	|	FH_d2.png
446	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_d3.png)	|	FH_d3.png
447	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_e1.png)	|	FH_e1.png
448	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/FH_e2.png)	|	FH_e2.png
449	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/HMT.png)	|	HMT.png
450	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_1.png)	|	SY_1.png
451	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_2.png)	|	SY_2.png
452	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_3.png)	|	SY_3.png
453	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_4.png)	|	SY_4.png
454	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_5.png)	|	SY_5.png
455	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_6.png)	|	SY_6.png
456	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_7.png)	|	SY_7.png
457	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/SY_e2.png)	|	SY_e2.png
458	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TGT.png)	|	TGT.png
459	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_1.png)	|	TG_1.png
460	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_10.png)	|	TG_10.png
461	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_11.png)	|	TG_11.png
462	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_12.png)	|	TG_12.png
463	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_13.png)	|	TG_13.png
464	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_14.png)	|	TG_14.png
465	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_2.png)	|	TG_2.png
466	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_3.png)	|	TG_3.png
467	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_4.png)	|	TG_4.png
468	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_5.png)	|	TG_5.png
469	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_6.png)	|	TG_6.png
470	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_7.png)	|	TG_7.png
471	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_8.png)	|	TG_8.png
472	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_9.png)	|	TG_9.png
473	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a1.png)	|	TG_a1.png
474	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a2.png)	|	TG_a2.png
475	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a3.png)	|	TG_a3.png
476	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a4.png)	|	TG_a4.png
477	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a5.png)	|	TG_a5.png
478	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_a6.png)	|	TG_a6.png
479	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b1.png)	|	TG_b1.png
480	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b2.png)	|	TG_b2.png
481	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b3.png)	|	TG_b3.png
482	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b4.png)	|	TG_b4.png
483	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b5.png)	|	TG_b5.png
484	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b6.png)	|	TG_b6.png
485	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_b7.png)	|	TG_b7.png
486	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c1.png)	|	TG_c1.png
487	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c10.png)	|	TG_c10.png
488	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c2.png)	|	TG_c2.png
489	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c3.png)	|	TG_c3.png
490	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c4.png)	|	TG_c4.png
491	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c5.png)	|	TG_c5.png
492	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c6.png)	|	TG_c6.png
493	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c7.png)	|	TG_c7.png
494	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c8.png)	|	TG_c8.png
495	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_c9.png)	|	TG_c9.png
496	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d1.png)	|	TG_d1.png
497	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d2.png)	|	TG_d2.png
498	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d3.png)	|	TG_d3.png
499	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d4.png)	|	TG_d4.png
500	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d5.png)	|	TG_d5.png
501	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_d6.png)	|	TG_d6.png
502	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_e1.png)	|	TG_e1.png
503	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_e2.png)	|	TG_e2.png
504	|	Ui	|	 ![](./Mods/Tiancai/Resources/Spr/Ui/TG_e3.png)	|	TG_e3.png
505	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/asd.png)	|	asd.png
506	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/chui1.png)	|	chui1.png
507	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/chui2.png)	|	chui2.png
508	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/dao1.png)	|	dao1.png
509	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/dao2.png)	|	dao2.png
510	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou.png)	|	fuzhou.png
511	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou1.png)	|	fuzhou1.png
512	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou2.png)	|	fuzhou2.png
513	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou3.png)	|	fuzhou3.png
514	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/fuzhou4.png)	|	fuzhou4.png
515	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/gong1.png)	|	gong1.png
516	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/gong2.png)	|	gong2.png
517	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/huan1.png)	|	huan1.png
518	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/huan2.png)	|	huan2.png
519	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jian.png)	|	jian.png
520	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jian1.png)	|	jian1.png
521	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jian2.png)	|	jian2.png
522	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jians1.png)	|	jians1.png
523	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jians2.png)	|	jians2.png
524	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jians3.png)	|	jians3.png
525	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jians4.png)	|	jians4.png
526	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/jians5.png)	|	jians5.png
527	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/qiang1.png)	|	qiang1.png
528	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/qiang2.png)	|	qiang2.png
529	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/shangqingdaopao.png)	|	shangqingdaopao.png
530	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/shangqingdaopao2.png)	|	shangqingdaopao2.png
531	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/taijidaopao.png)	|	taijidaopao.png
532	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/taijidaopao2.png)	|	taijidaopao2.png
533	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/taiqingdaopao.png)	|	taiqingdaopao.png
534	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/taiqingdaopao2.png)	|	taiqingdaopao2.png
535	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/wuzidaofu.png)	|	wuzidaofu.png
536	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/xianshenbaolu.png)	|	xianshenbaolu.png
537	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/yuqingdaopao.png)	|	yuqingdaopao.png
538	|	Wuqi	|	 ![](./Mods/Tiancai/Resources/Spr/Wuqi/yuqingdaopao2.png)	|	yuqingdaopao2.png
539	|	Wuqi	|	 ![](./"Mods/Tiancai/Resources/Spr/Wuqi/\351\255\224%20%282%29.png)"	|	224%20%282%29.png
540	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/dao.png)	|	dao.png
541	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/jian.png)	|	jian.png
542	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/mo.png)	|	mo.png
543	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/sanjie.png)	|	sanjie.png
544	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/sha.png)	|	sha.png
545	|	Ziti	|	 ![](./Mods/Tiancai/Resources/Spr/Ziti/zhan.png)	|	zhan.png



