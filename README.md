## 1. EDA
### Data Description
* `Name` : 게임의 이름
* `Platform` : 게임이 지원되는 플랫폼의 이름
* `Year` : 게임이 출시된 연도
* `Genre` : 게임의 장르
* `Publisher` : 게임을 제작한 회사
* `NA_Sales` : 북미지역에서의 출고량
* `EU_Sales` : 유럽지역에서의 출고량
* `JP_Sales` : 일본지역에서의 출고량
* `Other_Sales` : 기타지역에서의 출고량

#### ✨알고자 하는 것
1. 지역마다 선호하는 게임의 차이유무
2. 연도별 게임의 트렌드 변화
3. 출고량이 높은 게임에 대한 `분석 및 시각화 프로세스`

- 코드 작성 이전에 해야할 것

```python
# 그래프 한글패치
!sudo apt-get install -y fonts-nanum
!sudo fc-cache -fv
!rm ~/.cache/matplotlib -rf

import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
%config InlineBackend.figure_format = 'retina'

# 경고 무시
import warnings
warnings.filterwarnings('ignore')
```

#### Feature Engineering
- 사용된 Raw Data 및 Null Values

![Raw Data](https://user-images.githubusercontent.com/103639510/187372586-9bd57467-e7a4-477f-8356-cbf7b65fe278.png)
![Null Data](https://user-images.githubusercontent.com/103639510/187372733-510589cc-ed31-4ab0-8cad-34dc3e51fd90.png)

1. `Year`에 Null Values 존재: 총 판매량 100만을 넘기는 게임들은 직접 수정 & 나머지 Year는 같은 플랫폼의 최빈값(mode)로 대체, 처리 후 Integer 변환
2. `Genre`에 Null Values : Null Values 제거
3. `Publisher` : Unknown으로 대체
4. `Sales` : K와 M을 제거, K가 붙었던 데이터는 x 1/1000 으로 단위 변경 & float 변환
5. 지역별 `Sales`값을 전부 합쳐 **`Sales_sum`** 변수 생성

![preprocessing data](https://user-images.githubusercontent.com/103639510/187373873-9c91ebf2-6dd2-4cf2-8386-32293f197ed5.png)
---
## 2. Start Analysis

### 1. 상관분석 Use Heatmap
```python
colormap = plt.cm.PuBu
plt.figure(figsize = [12,8])
sns.heatmap(df_clean.iloc[:,5:10].astype(float).corr(), linewidths = 0.1, vmax = 1.0, square = True,
            cmap = colormap, linecolor = "white", annot = True, annot_kws = {"size" : 12})
plt.show()
```
<img width="544" alt="Heatmap" src="https://user-images.githubusercontent.com/103639510/187374445-69e24e2e-c828-43d5-a378-c5754c024569.png">

* 각 지역별 판매량에는 서로 상관성이 없음 
  * e.g) NA의 판매량이 오른다고 해서, EU의 판매량이 오르는 것은 아님

* 게임의 전체 판매량이 높은 NA와 EU가 총 판매량(Sales_sum)에 어느정도 영향을 줄 수 있음
  * 4개 지역 중 하나만 고른다면 NA 또는 EU를 고르는 것이 이득

---

### Q1. 장르에 따른 지역별(NA, EU, JP) 판매량의 평균을 비교해본다.(ANOVA 사용)
<br> 

- H_0: 지역에 따라 게임 장르에 대한 `판매량에 차이가 없다`.
- H_1: 지역에 따라 게임 장르에 대한 `판매량에 차이가 있다`.

#### 시각화 -> Barplot사용


1. ANOVA를 통해 Genre에 따라 게임판매량의 평균에 차이가 존재하는지 검정

![ANOVA](https://user-images.githubusercontent.com/103639510/187375611-73fc623a-fa5e-45e8-9235-c055a611b991.png)

- [x] NA부터 Others까지 전부 P_value가 0.05미만으로 매우 작으므로 모든 지역에서 Genre에 따라 게임판매량의 평균에 차이가 있다

2. ANOVA 사후검정
- 각 지역별로 선호하는 장르가 무엇인지 시각화(Barplot)

<img src="https://user-images.githubusercontent.com/103639510/187376264-d56647cf-07dc-4020-a60e-c64191790f87.png" width = "1100">

>* NA, EU, Others는 Action 장르의 게임을 가장 선호하는 것으로 확인할 수 있다.
>  * 뒤이어 Shooter 및 Sports가 선호되는 장르에 해당된다.
>* JP는 그 외 지역과 달리 Role Playing 즉, RPG장르를 가장 선호하고 있고, 오히려 Shooter 장르는 하위권에 속하는 것을 알 수 있다.
>  * 이어서 Action 및 Sports 장르가 선호됨을 알 수 있다.


- <span style = "font-size:150%"> 전체적으로 Action 장르가 많은 인기를 끌고 있음 </span>


### Q2. 연도별 게임의 트렌드가 존재하는가?
1. groupby('Year')를 사용해서 연도별 게임의 판매량 및 장르를 확인
2. 각 지역별 판매량을 연도별로 시각화(Use LIneplot, x = Year, y = sales)

<img width="1169" alt="지역별 게임 판매량" src="https://user-images.githubusercontent.com/103639510/187380861-7d3b6045-78de-4188-bd6e-ef7d20c08180.png">

- NA > EU > Others > JP 순으로 게임 판매량의 최댓값에 차이가 남

#### 1. 연도별로 어느 장르의 게임이 많이 팔렸는지 시각화(Lineplot)

<img width="1057" alt="연도별 게임 장르 트렌드(출고량)" src="https://user-images.githubusercontent.com/103639510/187381370-f7a6a518-8f8a-4645-801b-f487a3d28f4b.png">

> * 언뜻 보면, Action 장르의 게임이 인기 많은 걸로 보이기는 하나 1년 단위로 꺾이면서, 그래프가 너무 엉켜있어 쉽게 판단하기 힘듬
>   * 특히, 80~90년도는 여러장르들이 너무 겹쳐서 알기가 매우 힘듬

* 따라서, `5년 단위`로 x축의 변수를 새로 정의하고, 게임 판매량이 아닌 그 연도에 얼마나 판매되었는지 `비율(Ratio)`를 새로 **`Feature Engineering`** 한다.


```Python
ratio = []

for i in range(len(df_trend_sum)):
  for j in range(df_trend_count[df_trend_count.Year == df_trend_sum.index[i]].shape[0]):
    ratio.append(df_trend_count[df_trend_count.Year == df_trend_sum.index[i]].loc[:,'Sales_sum'].iloc[j] / df_trend_sum.iloc[i])

# 각 연도에 따른 장르별 판매비율에 대한 데이터프레임
```

<details>
<summary>Feature Engineering data </summary>

<img width = "340" height = "330" src = "https://user-images.githubusercontent.com/103639510/187406574-e821d79b-a181-45ee-b33d-c9a94f5c1da6.png">

</details>
<img width="760" height = "400" alt="5년 단위 장르" src="https://user-images.githubusercontent.com/103639510/187406678-3db82b6d-4b03-4a4a-8ae6-31a77b2cefc6.png">

>* 위의 그래프를 통해서 86년도 이전부터 95년도 까지는 `Platform` 장르의 게임이 가장 많은 비율로 판매되었음을 알 수 있다.
>  * 96~00년도에는 `Role Playing` 장르가 근소한 차이로 가장 높지만, 이후 01년도부터 현재까지는 `Action` 장르의 게임이 가장 많은 비율로 판매됨을 알 수 있음.
>* 추가적으로 00년도 이후부터 게임장르의 트렌드에 대한 변화가 적게 나타나는 것처럼 보이기 때문에 이후 분석은 00년도 이후에 대한 분석으로 **`최근 20년간`** 의 게임 트렌드를 분석

#### 2. 연도별로 어떤 플랫폼을 지원하는 게임이 잘 팔렸는지 시각화(stackplot)

1. 지원 플랫폼의 종류가 너무 많을 뿐 아니라, 세대가 변화되는 플랫폼들이 다수 존재하기 때문에 크게 `Nintendo, Xbox, PS, PC, Others`로 나눈다.
2. 크게 나눈 Categorical 변수들을 Pivot_table로 바꾼다.
<details>
<summary>Pivot_table Data </summary>
 <img width="520" src="https://user-images.githubusercontent.com/103639510/187599775-3d74c251-157d-4c4e-9aae-a8d8909ae5f7.png">

 <details>
 <summary> Code </summary>
 
 ```
 NA_kind = platform_kind.groupby(['Platform_kind','Year'], as_index = False).NA_Sales.mean()

 NA_pivot = pd.pivot_table(NA_kind,
               index = 'Year',
               columns = 'Platform_kind',
               values = 'NA_Sales'
               ).fillna(0)
 NA_pivot.head()
 
 EU_kind = platform_kind.groupby(['Platform_kind','Year'], as_index = False).EU_Sales.mean()

 EU_pivot = pd.pivot_table(EU_kind,
               index = 'Year',
               columns = 'Platform_kind',
               values = 'EU_Sales'
               ).fillna(0)
 EU_pivot.head()
 
 JP_kind = platform_kind.groupby(['Platform_kind','Year'], as_index = False).JP_Sales.mean()

 JP_pivot = pd.pivot_table(JP_kind,
               index = 'Year',
               columns = 'Platform_kind',
               values = 'JP_Sales'
               ).fillna(0)
 JP_pivot.head()
 
 other_kind = platform_kind.groupby(['Platform_kind','Year'], as_index = False).Other_Sales.mean()

 other_pivot = pd.pivot_table(other_kind,
                             index = 'Year',
                             columns = 'Platform_kind',
                             values = 'Other_Sales').fillna(0)
 other_pivot.head()
 
 Sum_kind = platform_kind.groupby(['Platform_kind','Year'], as_index = False).Sales_sum.mean()

 Sum_pivot = pd.pivot_table(Sum_kind,
               index = 'Year',
               columns = 'Platform_kind',
               values = 'Sales_sum'
               ).fillna(0)

 Sum_pivot.head()
 ```
 </details>
</details>
<img width="1381" src="https://user-images.githubusercontent.com/103639510/187600113-7f530fe4-2935-4db2-8e99-dbff04412b6d.png">

>* 북미의 경우, 10년도 이후 Xbox가 한번 트렌드가 되었고, 현재에는 Nintendo 플랫폼의 판매량이 늘어나는 추세이다.
>* 유럽(EU)의 경우, 12년도 이후 Xbox의 판매량이 급증하게 되었음을 확인할 수 있는데, 이를 통해 Xbox플랫폼을 지원하는 게임의 발매가 기대됨
>* 일본(JP)의 경우, Nintendo가 압도적으로 많이 판매되기 때문에 거의 매년 Nintendo 플랫폼이 가장 트렌드임을 알 수 있음
>* 기타 지역의 경우, 12년도 이후 Xbox 플랫폼을 지원하는 게임의 판매량이 크게 늘었고, 기타 플랫폼(Others) 지원 게임의 판매량이 높은 것을 보면 Xbox를 주축으로 기타 플랫폼을 지원하는 게임들이 인기가 있을거라고 추측됨

<img width="1040" src="https://user-images.githubusercontent.com/103639510/187603265-bc64809c-eada-4ffd-8956-6b123263f462.png">

>* 전 세계에서 지원 플랫폼에 따른 게임 판매량을 확인해 보면 Xbox와 Nintendo가 트렌드임을 알 수 있다.
>* 하지만, 17년 이후의 게임 판매량에 관한 데이터가 적기 때문에 이를 통해 어떤 플랫폼을 지원하면 유리한지는 확실하게 알기 힘듬.
>* 특히, 그래프를 보면 PS 또한 Nintendo 와 비슷할 정도로 많은 판매량을 유지하고 있음을 알 수 있다. PS는 근래에 PSV를 출시하면서 더욱 주요 플랫폼에 자리잡을 가능성이 예상되는데 현재 데이터만으로는 22년도의 지원 플랫폼의 트렌드를 판단하기 힘듬
* 따라서, 16년도 이후의 게임 데이터를 새로 얻어서 재분석이 요망됨
* 또한, 12~14년을 기점으로 전체적인 판매량이 낮아지는 이유 또한 스마트폰이 보급된 시기와 겹치게 되므로 모바일 게임산업에 관한 데이터가 추가적으로 필요함


---
### Q3. 출고량이 높은 게임들에 대해서 분석

* 00년도 이후의 데이터로 분석

>* 전체 판매량(Sales_sum)에서 인기가 많은 게임들에 대한 비율을 원그래프로 표현
>* 지역별 판매량에서 인기가 많은 게임들에 대한 비율을 원그래프로 표현

<details>
<summary> Global </summary>

<img width="450" alt="Global Top10 Games" src="https://user-images.githubusercontent.com/103639510/187595708-61149f39-89f2-4d4d-b0ff-0d0b97c004c1.png">

</details>
<details>
<summary> Each Region </summary>

<img width="1050" src="https://user-images.githubusercontent.com/103639510/187595505-f300ef95-1c23-4f15-9c1e-96bbd06c203b.png">

</details>
* 세계 전체로 살펴보면 Wii Sports가 가장 인기가 많고 그에 뒤따라서 GTA5가 인기가 많은 것으로 확인된다.
* NA 및 EU가 00년도 이후 Wii Sports 게임이 가장 인기가 많음
* JP의 경우, Super Mario Bros가 가장 인기가 있지만 Top5 내의 게임들의 판매비중의 차이가 근소하고 오히려 RPG 장르인 Pokemon 시리즈의 게임들이 많은 비중을 차지한다.

---

### **결론**
#### 게임을 개발하는 연도가 2017년이었을 경우

>1. 현재 게임 장르의 트렌드는 `Action`장르의 게임이 가장 주를 이루고 있으며, `Shooter`와 `Sports`가 그 뒤를 잇고 있음
>2. 일본은 예외적으로 `Role Playing(RPG)` 장르가 가장 인기가 많으며, `Shooter` 장르의 경우는 오히려 비인기 장르에 해당된다.
>3. 지원 플랫폼은 전체적으로 Nintendo와 Xbox를 지원하는 게임의 선호도가 가장 큼

#### 게임을 현재(2022년) 개발할 경우

>1. 장르의 변화는 00년도 이후 크게 바뀌는 추세가 아니기 때문에 일본을 제외한다면, `Action` 및 `Sports`장르의 게임을 제작해도 좋을 것이라고 판단됨
>2. 지원 플랫폼의 경우, 게임의 전체 판매량 자체가 12~14년을 기준으로 점차 떨어지고 있기 때문에 모바일 분야에서의 게임 개발을 고려해 봐야할 필요가 있음
>3. 콘솔 게임을 개발하는 경우, Nintendo도 용이하지만 현재 새로운 세대의 게임기 PSV의 영향력을 무시할 수 없으므로 16년도 이후의 게임 판매량 데이터를 새로 얻을 필요가 있음
