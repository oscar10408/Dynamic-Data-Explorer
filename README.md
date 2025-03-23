# 🧠 Interactive Visualization of School Demographics and Performance

This project explores the relationship between school demographics, economic status, and academic performance using interactive visualizations. It builds upon previous static charts and enhances the user experience by allowing dynamic data exploration through sliders, filters, and interactive selections.

---

## 📷 Demo
<img src="https://github.com/oscar10408/Dynamic-Data-Explorer/blob/main/images/Interactive_chart_1.gif" alt="demo" width="1000" height="400"/>
<img src="https://github.com/oscar10408/Dynamic-Data-Explorer/blob/main/images/Interactive_chart_2.gif" alt="demo" width="1000" height="400"/>

---

## 📌 Project Overview

This visualization investigates:

- How **economic status** (measured by free lunch eligibility) relates to **academic performance** (mean scale score).
- Whether **school size** influences **racial composition** and **gender distribution**.
- How **geographic distribution** affects demographic patterns in New York schools.

The interactive elements empower users to explore these variables at multiple levels with real-time feedback.

---

## 📊 Key Features

### 🔹 Visualization 1: Scatter Plot + Demographic Bar Chart
- **X-axis**: Free Lunch Eligibility (%)
- **Y-axis**: Mean Scale Score
- **Point Size**: Total student population
- **Interactive sliders** to filter schools by size
- **Linked bar chart** updates to show racial breakdown of selected schools

```python
max_student_slider = alt.binding_range(
    name='Max_Total_Students',
    min=0, 
    max=1409,
    step=50,
    )

min_student_slider = alt.binding_range(
    name='Min_Total_Students',
    min=0, 
    max=1409,
    step=50,
)

selector_max = alt.selection_point(
    bind= max_student_slider,
    fields= ["Max_Total_Students"],
    value=1409
    )

selector_min = alt.selection_point(
    bind= min_student_slider,
    fields= ["Min_Total_Students"],
    value=0
    )

base = (
    alt.Chart(df)
    .transform_aggregate(
        fl='mean(free_lunch_eligible)',
        sc='mean(mean_scale_score)',
        wr='mean(white_ratio)',
        White='mean(white_students)',
        Black='mean(black_students)',
        Asian='mean(asian_students)',
        tot='max(total_students)',
        groupby=['school_name']  
    )
    .add_params(selector_max, selector_min)
    .transform_filter(alt.datum.tot > selector_min.Min_Total_Students)
    .transform_filter(alt.datum.tot < selector_max.Max_Total_Students)
)



brush = alt.selection_interval(encodings=['x','y'], empty=True)


scatter_plot_interactive = (
    base.mark_circle(fill=None, strokeWidth=1.5)
    .encode(
        x=alt.X('fl:Q', title='Free Lunch Eligible', scale=alt.Scale(domain=[-20, 1000])),
        y=alt.Y('sc:Q', title='Mean Scale Score', scale=alt.Scale(domain=[300, 650])),
        stroke=alt.Stroke(
            'wr:Q', 
            scale=alt.Scale(scheme="reds"), 
            legend=alt.Legend(title='White Ratio')
        ),
        size=alt.Size('tot:Q', legend=None),
        tooltip=[
            alt.Tooltip('school_name:N', title='School'),
            alt.Tooltip('sc:Q', title='Mean Score', format='.3f'),
            alt.Tooltip('fl:Q', title='Free Lunch', format='.1f'),
            alt.Tooltip('wr:Q', title='White Ratio', format='.3f'),
            alt.Tooltip('tot:Q', title='tot', format='.3f')
        ]
    )
    .add_params(brush)
    .properties(
        width=600,
        height=400,
        title=alt.TitleParams('Financial vs. Score (by School)', fontSize=20)
    )
)


bar_chart_interactive = (
    base
    .transform_filter(brush)
    .transform_fold(
        ['White', 'Black', 'Asian'], 
        as_=['race', 'count']
    )
    .mark_bar()
    .encode(
        x=alt.X(
            'school_name:N',
            sort=alt.EncodingSortField(field='tot', op='max', order='descending'),
            title='School',
            axis = None
        ),
        y=alt.Y('count:Q', title='Proportion of Races'),
        color=alt.Color('race:N', legend=alt.Legend(title='Race')),
        order=alt.Order('count:Q', sort='ascending'),
        tooltip=[
            alt.Tooltip('school_name:N', title='School'),
            alt.Tooltip('race:N', title='Race'),
            alt.Tooltip('count:Q', title='Count')
        ]
    )
    .properties(
        width=600,
        height=400,
        title=alt.TitleParams('Bar Chart: Composition of Selected Schools', fontSize=20)
    )
)

final_chart_1 = alt.hconcat(scatter_plot_interactive, bar_chart_interactive).configure_view(strokeWidth=0)
final_chart_1
```

### 🔹 Visualization 2: New York School Map + Gender Pyramid
- **Map of NYC** showing school locations, with:
  - Point size = school size
  - Color = white student ratio
- **Min White Ratio Slider**: filters schools by white population percentage
- **Dropdown filter**: select dominant racial group (Black, White, Asian, etc.)
- **Gender Pyramid Chart**: updates based on selected schools from map

```python
brush = alt.selection_interval(encodings=['longitude','latitude'])


white_slider = alt.param(
    name='MinWhitePct',
    bind=alt.binding_range(
        min=0, 
        max=max(df2['White_Percentage']), 
        step=0.05, 
        name='Min White %'),
    value=0
)

race_select = alt.param(
    name="SelectedRace",
    bind=alt.binding_select(
        options=["All", "Asian", "Black", "White", "Hispanic", "Native"],
        name="Main Race" 
    ),
    value="All"
)

base = (alt.Chart(df2).add_params(brush, white_slider, race_select)
    ).transform_filter(f"datum.White_Percentage >= {white_slider.name}"
    ).transform_filter(f"datum.Main_race == {race_select.name} || {race_select.name} == 'All'"
    )


states = alt.topo_feature(data.us_10m.url, feature='states')

# Background geoshape
background = (
    alt.Chart(states)
    .mark_geoshape(fill='lightgray', stroke='white')
    .project(type='mercator', center=[-74.5, 40.7], scale=40000)
)

points = (
    base
    .mark_circle(color='red')
    .encode(
        longitude='Longitude:Q',
        latitude='Latitude:Q',
        size=alt.Size('total_students:Q', title='Sclae of School', 
                      legend=alt.Legend(
                             orient='none',
                             direction='vertical',
                             legendX=10,
                             legendY=10)),
        color=alt.Color('White_Percentage:Q', scale=alt.Scale(scheme='reds'),
                        legend=alt.Legend(
                               orient='top-right',
                               direction='vertical',)),
        tooltip=['school_name:N','total_students:Q','White_Percentage:Q']
    )
    .project(type='mercator', center=[-74.5, 40.7], scale=400)
)

map_chart = (background + points).properties(width=800, height=600)

pyramid_chart = (
    base
    .transform_fold(['Male_Students','Female_Students'], as_=['gender','count'])
    .transform_calculate("count2", "datum.gender == 'Male_Students' ? -datum.count : datum.count")
    .transform_filter(brush)
    .mark_bar()
    .encode(
        longitude='Longitude:Q',
        latitude='Latitude:Q',
        y=alt.Y('school_name:N', sort='-x', axis=None),
        x=alt.X('count2:Q', title=None, axis=alt.Axis(labelExpr="abs(datum.value)")),
        color=alt.Color(
            'gender:N',
            scale=alt.Scale(domain=['Male_Students','Female_Students'],
                            range=['#1F77B4','#D62728']),
        ),
        tooltip=['school_name:N','gender:N','count:Q']
    )
    .properties(width=800, height=600)
)

final_chart_2 = alt.hconcat(
    map_chart.properties(title=alt.TitleParams("White Students Percentage of School", fontSize=20)),
    pyramid_chart.properties(title=alt.TitleParams("Gender Composition", fontSize=20))
).configure_view(strokeWidth=0)


final_chart_2
```

---

## 🛠 Technologies Used

- 📒 Jupyter Notebook
- 📈 Altair (for interactive charts)
- 🐼 Pandas (data handling)
- 🔍 Vega-Lite (underlying Altair rendering)

---

