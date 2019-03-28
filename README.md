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
