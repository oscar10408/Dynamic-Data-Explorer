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

### 🔹 Visualization 2: New York School Map + Gender Pyramid
- **Map of NYC** showing school locations, with:
  - Point size = school size
  - Color = white student ratio
- **Min White Ratio Slider**: filters schools by white population percentage
- **Dropdown filter**: select dominant racial group (Black, White, Asian, etc.)
- **Gender Pyramid Chart**: updates based on selected schools from map

---

## 🛠 Technologies Used

- 📒 Jupyter Notebook
- 📈 Altair (for interactive charts)
- 🐼 Pandas (data handling)
- 🔍 Vega-Lite (underlying Altair rendering)

---

