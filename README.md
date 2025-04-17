# U.S. Economic Power Dashboard - Shiny App

This app provides an interactive dashboard comparing U.S. economic power against global metrics, including stock price visualizations for selected U.S. companies.

---

## 💡 Features

- **Landing page** with custom branding (flag + power logos)
- **Interactive bar plots** comparing U.S. vs Global values across:
  - GDP
  - Consumer Spending
  - FDI
  - Innovation Index
- **Candlestick stock charts** for selected NYSE stocks
- **Custom summary text** explaining each selected metric
- **Animated transition** from landing page to main dashboard

---

## 📁 App Structure

```
/your-app-folder/
├── app.R              # main Shiny app file (this file)
├── www/
│   ├── styles.css     # your custom stylesheet
│   ├── flag-logo.png  # U.S. logo for the dashboard header
│   └── power-logo.png # Power logo for the landing page
```

Ensure the `www/` folder exists and contains the above assets for everything to render properly.

---

## 🚀 Launch Instructions

1. Place `app.R` and a `www/` folder with `styles.css`, `flag-logo.png`, and `power-logo.png` in the same directory.
2. Run the app using `runApp()` or click **Run App** in RStudio.

---

## 🛠️ CSS Behavior Notes

This snippet in `app.R` ensures that the main dashboard is initially hidden until the **"Enter Dashboard"** button is clicked:

```r
# Inline CSS
  tags$style(HTML("\n    #main-app { display: none; }\n    #main-app.visible { display: block !important; }\n  "))
```

And then `shinyjs` handles the toggle between landing and main dashboard views.

---

## 📈 Future Ideas

- Add timeline animation or slider for historical stock data
- Embed mini-map showing FDI inflows by region
- Add KPI cards with small multiples

---

Built with 💼 `shiny`, `plotly`, `quantmod`, and `ggplot2`

---

### Example: How to add a KPI summary box style

In your `styles.css`:
```css
.summary-box {
  background-color: #f9f9f9;
  border-left: 4px solid #002868;
  padding: 10px 15px;
  margin-top: 15px;
  font-family: 'Roboto', sans-serif;
  font-size: 14px;
  color: #333;
}
```

And in your `mainPanel()` section:
```r
  div(
    class = "summary-box",
    verbatimTextOutput("summaryText")
  )
```

Enjoy building! 🇺🇸📊
