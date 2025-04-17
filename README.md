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
│   ├── styles.css     # custom stylesheet
│   ├── index.html     # landing page HTML (optional, if using HTML-based landing)
│   ├── flag-logo.png  # U.S. logo for the dashboard header
│   └── power-logo.png # Power logo for the landing page
```

Ensure the `www/` folder exists and contains the above assets for everything to render properly. If using the `div`-based landing page (as in the latest code), `index.html` is not required.

---

## 🚀 Launch Instructions

1. Place `app.R` and a `www/` folder with `styles.css`, `flag-logo.png`, and `power-logo.png` in the same directory.
2. If using an HTML-based landing page, include `index.html` in the `www/` folder.
3. Install required R packages:
   ```R
   install.packages(c("shiny", "shinyjs", "ggplot2", "quantmod", "plotly"))
   ```
4. Run the app using `runApp()` or click **Run App** in RStudio.

---

## 🛠️ CSS Behavior Notes

This snippet in `app.R` ensures that the main dashboard is initially hidden until the **"Enter Dashboard"** button is clicked:

```R
# Inline CSS
tags$style(HTML("
  #main-app { display: none; }
  #main-app.visible { display: block !important; }
"))
```

And then `shinyjs` handles the toggle between landing and main dashboard views.

---

## 📈 Future Ideas

- Add timeline animation or slider for historical stock data
- Embed mini-map showing FDI inflows by region
- Add KPI cards with small multiples

---

## 📝 Code and Assets

### `app.R`

The main Shiny app file, implementing the dashboard with a landing page, economic metrics, and stock price visualizations.

```R
library(shiny)
library(shinyjs)
library(ggplot2)
library(quantmod)
library(plotly)

# Add resource path for www/ folder
addResourcePath("static", "www")

# ---- UI ----
ui <- fluidPage(
  useShinyjs(),
  
  tags$head(
    tags$link(rel = "stylesheet", type = "text/css", href = "static/styles.css"),
    tags$title("U.S. Economic Power Dashboard"),
    # Inline CSS to ensure main-app visibility
    tags$style(HTML("
      #main-app { display: none; }
      #main-app.visible { display: block !important; }
    "))
  ),
  
  # Landing page
  div(id = "landing",
      div(class = "landing-container",
          img(src = "static/power-logo.png", class = "landing-logo", alt = "Power Logo"),
          h1("U.S. Economic Power Dashboard", class = "landing-title"),
          actionButton("launchApp", "Enter Dashboard", class = "launch-button")
      )
  ),
  
  # Main App UI (initially hidden)
  hidden(
    div(id = "main-app",
        titlePanel(
          div(
            img(src = "static/flag-logo.png", height = 40, alt = "U.S. Economic Power Logo", style = "margin-right: 10px; vertical-align: middle;"),
            "U.S. Economic Power Dashboard"
          )
        ),
        sidebarLayout(
          sidebarPanel(
            selectInput("metric", "Choose a metric:",
                        choices = c("GDP", "Consumer Spending", "FDI", "Innovation Index", "Stock Prices"),
                        selected = "GDP"),
            conditionalPanel(
              condition = "input.metric == 'Stock Prices'",
              selectInput(
                inputId = "stock",
                label = "Select NYSE Stock",
                choices = c("Apple (AAPL)" = "AAPL", 
                            "IBM (IBM)" = "IBM", 
                            "Microsoft (MSFT)" = "MSFT", 
                            "JPMorgan Chase (JPM)" = "JPM"),
                selected = "AAPL"
              ),
              dateRangeInput(
                inputId = "date",
                label = "Date Range",
                start = Sys.Date() - 365,
                end = Sys.Date(),
                min = "2007-01-01",
                max = Sys.Date(),
                format = "yyyy-mm-dd"
              )
            ),
            actionButton("returnLanding", "Return to Landing Page")
          ),
          mainPanel(
            plotlyOutput("mainPlot"),
            div(class = "summary-box", verbatimTextOutput("summaryText"))
          )
        )
    )
  )
)

# ---- Server ----
server <- function(input, output, session) {
  # Handle launch button click
  observeEvent(input$launchApp, {
    print("Launch button clicked")
    # Hide landing and show main-app
    shinyjs::hide("landing")
    shinyjs::show("main-app")
    # Add visible class and force display
    shinyjs::addClass("main-app", "visible")
    # Debug in browser console
    session$sendCustomMessage(type = "log", message = "Main app shown")
    # Force UI refresh
    shinyjs::runjs("document.getElementById('main-app').style.display = 'block';")
  })
  
  # Handle return to landing page
  observeEvent(input$returnLanding, {
    print("Return to landing clicked")
    shinyjs::show("landing")
    shinyjs::hide("main-app")
    shinyjs::removeClass("main-app", "visible")
    session$sendCustomMessage(type = "log", message = "Landing page shown")
  })
  
  # Custom message handler for browser console logging
  observe({
    session$onFlushed(function() {
      session$sendCustomMessage(type = "log", message = "Session initialized")
    }, once = TRUE)
  })
  
  # Sample data (expanded for U.S. vs Global)
  dataset <- data.frame(
    metric = rep(c("GDP", "Consumer Spending", "FDI", "Innovation Index"), each = 2),
    region = rep(c("U.S.", "Global"), times = 4),
    value = c(25, 70, 18, 45, 10, 40, 9, 7),
    label = rep(c("Trillions USD", "Trillions USD", "Trillions USD", "Index Score"), each =  prehensive list, but be cautious of UI clutter.
  )
  
  # Reactive to fetch stock data
  stockData <- reactive({
    if (input$metric == "Stock Prices") {
      req(input$stock, input$date)
      tryCatch({
        getSymbols(input$stock, 
                   from = input$date[1], 
                   to = input$date[2], 
                   auto.assign = FALSE)
      }, error = function(e) {
        NULL
      })
    } else {
      NULL
    }
  })
  
  # Render Plot
  output$mainPlot <- renderPlotly({
    req(input$metric)
    
    if (input$metric == "Stock Prices") {
      data <- stockData()
      if (is.null(data)) {
        return(plot_ly() %>% 
                 layout(title = "Error: Could not fetch stock data"))
      }
      # Convert xts to data frame
      df <- data.frame(
        Date = index(data),
        Open = as.numeric(Op(data)),
        High = as.numeric(Hi(data)),
        Low = as.numeric(Lo(data)),
        Close = as.numeric(Cl(data))
      )
      plot_ly(data = df, 
              x = ~Date,
              open = ~Open,
              high = ~High,
              low = ~Low,
              close = ~Close,
              type = "candlestick",
              name = input$stock) %>%
        layout(
          title = paste(input$stock, "Stock Price"),
          xaxis = list(title = "Date"),
          yaxis = list(title = "Price (USD)"),
          showlegend = FALSE
        )
    } else {
      selected <- dataset[dataset$metric == input$metric, ]
      ggplot_plot <- ggplot(selected, aes(x = region, y = value, fill = region)) +
        geom_bar(stat = "identity", position = "dodge") +
        scale_fill_manual(values = c("U.S." = "#002868", "Global" = "#B22234")) +
        theme_minimal() +
        labs(
          title = paste("U.S. vs Global in", input$metric),
          y = selected$label[1],
          x = NULL
        ) +
        theme(legend.position = "top")
      ggplotly(ggplot_plot)
    }
  })
  
  # Render Text
  output$summaryText <- renderPrint({
    req(input$metric)
    if (input$metric == "Stock Prices") {
      data <- stockData()
      if (is.null(data)) {
        "Error: Could not fetch stock data"
      } else {
        latest_price <- as.numeric(Cl(data)[nrow(data)])
        paste("The latest closing price for", input$stock, "is: $", round(latest_price, 2))
      }
    } else {
      selected <- dataset[dataset$metric == input$metric & dataset$region == "U.S.", ]
      paste("The U.S.", input$metric, "value is:", selected$value, selected$label)
    }
  })
}

# ---- Launch ----
shinyApp(ui, server)
```

### `www/styles.css`

The stylesheet for the app, including styles for the landing page, buttons, and the summary box as suggested in the README. These styles are designed to match the U.S. theme and ensure a polished look.

```css
/* General styling */
body {
  font-family: 'Roboto', sans-serif;
  background-color: #f4f4f4;
  color: #333;
}

/* Landing page container */
.landing-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  height: 100vh;
  background: linear-gradient(135deg, #002868, #B22234);
  color: white;
  text-align: center;
  padding: 20px;
}

/* Landing page logo */
.landing-logo {
  max-width: 150px;
  margin-bottom: 20px;
}

/* Landing page title */
.landing-title {
  font-size: 2.5em;
  font-weight: bold;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

/* Launch button */
.launch-button {
  background-color: #B22234;
  color: white;
  border: none;
  padding: 12px 24px;
  font-size: 1.2em;
  font-weight: bold;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.launch-button:hover {
  background-color: #8B1A1A;
}

/* Main app styling */
.titlePanel {
  background-color: #002868;
  color: white;
  padding: 10px;
  border-radius: 5px;
}

/* Sidebar styling */
.sidebar {
  background-color: #ffffff;
  border: 1px solid #ddd;
  border-radius: 5px;
  padding: 15px;
}

/* Summary box for verbatimTextOutput */
.summary-box {
  background-color: #f9f9f9;
  border-left: 4px solid #002868;
  padding: 10px 15px;
  margin-top: 15px;
  font-family: 'Roboto', sans-serif;
  font-size: 14px;
  color: #333;
}

/* Plotly chart container */
.plotly {
  margin-top: 20px;
}

/* Ensure main-app is hidden initially (reinforced by inline CSS) */
#main-app {
  display: none;
}
```

### `www/index.html` (Optional)

This is a sample `index.html` file that replicates the current `div`-based landing page (logo, title, and "Enter Dashboard" button) for cases where you want to use an HTML-based landing page (as in earlier versions of your app). If you have a specific `index.html`, replace this with your version. This file works with the `tags$iframe` approach from earlier responses.

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>U.S. Economic Power Dashboard</title>
  <link rel="stylesheet" href="styles.css">
  <style>
    body {
      margin: 0;
      padding: 0;
      font-family: 'Roboto', sans-serif;
    }
  </style>
</head>
<body>
  <div class="landing-container">
    <img src="power-logo.png" class="landing-logo" alt="Power Logo">
    <h1 class="landing-title">U.S. Economic Power Dashboard</h1>
    <button class="launch-button" onclick="Shiny.setInputValue('launchApp', Math.random());">Enter Dashboard</button>
  </div>
  <script src="https://cdn.jsdelivr.net/npm/shiny@1.7.5/dist/shiny.min.js"></script>
</body>
</html>
```

**Notes on `index.html`**:
- The `src="power-logo.png"` assumes the logo is in the `www/` folder and accessible via `static/power-logo.png` due to `addResourcePath`.
- The `onclick="Shiny.setInputValue('launchApp', Math.random());"` triggers the `input$launchApp` event in Shiny, matching the `observeEvent(input$launchApp, ...)` in `app.R`.
- The `<script>` tag includes Shiny's JavaScript library for `Shiny.setInputValue`. If using this file, you may need to adjust the `src` path or bundle Shiny locally.

To use this `index.html` instead of the `div`-based landing page, modify the UI in `app.R` to:
```R
# Landing page
div(id = "landing",
    tags$iframe(
      src = "static/index.html",
      width = "100%",
      height = "100vh",
      style = "border: none;"
    )
),
```

### Modifications to README and Code

1. **Updated `app.R`**:
   - Included the latest working version with the `xts`-to-data-frame fix for the stock graph.
   - Added the `.summary-box` class to the `verbatimTextOutput` in the `mainPanel`:
     ```R
     div(class = "summary-box", verbatimTextOutput("summaryText"))
     ```
   - Kept the `div`-based landing page (not `index.html`) as per your latest code, but provided `index.html` as an optional alternative.
   - Removed the commented-out `shinyjs::delay` for simplicity, but you can re-enable it if desired:
     ```R
     shinyjs::delay(5000, {
       shinyjs::hide("landing")
       shinyjs::show("main-app")
       shinyjs::addClass("main-app", "visible")
       session$sendCustomMessage(type = "log", message = "Main app shown after delay")
     })
     ```

2. **Created `styles.css`**:
   - Added styles for `landing-container`, `landing-logo`, `landing-title`, and `launch-button` to match the landing page's appearance.
   - Included the `.summary-box` style from the README example.
   - Added general styles for the main app (e.g., `.titlePanel`, `.sidebar`, `.plotly`) to ensure a cohesive look.
   - Reinforced `#main-app` hiding to align with inline CSS.

3. **Provided `index.html`**:
   - Created a sample HTML file replicating the current landing page for flexibility.
   - Ensured compatibility with `styles.css` and Shiny's JavaScript for `input$launchApp`.

4. **Updated README**:
   - Included all code files (`app.R`, `styles.css`, `index.html`) as requested.
   - Clarified the app structure, noting that `index.html` is optional if using the `div`-based landing page.
   - Added package installation instructions.
   - Kept the original features, structure, launch instructions, CSS notes, future ideas, and example intact.

### Testing Instructions

1. **Set Up the Directory**:
   - Create a folder (e.g., `us-economic-dashboard/`).
   - Save `app.R` in the root.
   - Create a `www/` subfolder and save `styles.css`, `flag-logo.png`, `power-logo.png`, and (optionally) `index.html`.
   - Ensure `flag-logo.png` and `power-logo.png` are valid images (e.g., PNG files with U.S. flag and power-themed logos).

2. **Run the App**:
   - Open `app.R` in RStudio.
   - Install dependencies: `install.packages(c("shiny", "shinyjs", "ggplot2", "quantmod", "plotly"))`.
   - Click **Run App** or run `shiny::runApp()`.

3. **Verify Functionality**:
   - **Landing Page**: Confirm the logo, title, and "Enter Dashboard" button display, and clicking the button transitions to the main app.
   - **Main App**: Check that the sidebar dropdown shows all metrics, including "Stock Prices."
   - **Stock Prices**: Select "Stock Prices," choose a stock (e.g., AAPL), and verify the candlestick chart renders without errors.
   - **Other Metrics**: Test GDP, Consumer Spending, etc., to ensure bar plots and summary text (in the `.summary-box`) display correctly.
   - **Return to Landing**: Click "Return to Landing Page" to confirm the transition back.

4. **Debugging**:
   - Check the R console for `print("Launch button clicked")` or stock data errors.
   - Open the browser console (F12, Console tab) for messages like "Main app shown."
   - Verify resources (`static/styles.css`, `static/power-logo.png`, etc.) load without 404 errors in the browser's Network tab.

### Notes

- **Optional `index.html`**: The `div`-based landing page in `app.R` is the current implementation. If you prefer the `index.html` approach, update the UI as noted above and ensure `www/index.html` exists.
- **Stock List Expansion**: To include more NYSE stocks, use:
  ```R
  nyse_stocks <- stockSymbols(exchange = "NYSE")
  stock_choices <- setNames(nyse_stocks$Symbol, paste(nyse_stocks$Name, "(", nyse_stocks$Symbol, ")"))
  ```
  Update the `selectInput` for `stock` to use `choices = stock_choices`.
- **CSS Customization**: Adjust `styles.css` colors, fonts, or layouts to match your branding. The current palette uses U.S. flag colors (`#002868` navy, `#B22234` red).
- **Deployment**: For shinyapps.io, ensure the `www/` folder is included, and all packages are installed. Test stock data fetching, as it requires an internet connection.

### Relevant Sources

- Shiny documentation on `shinyjs` and `plotly` ().
- Quantmod documentation for stock data ().
- Plotly R candlestick chart examples ().
- Stack Overflow on Shiny UI transitions and data frame errors ().

If you encounter issues (e.g., resource loading errors, styling problems, or stock data fetch failures), please share:
- Specific error messages (R console or browser console).
- Contents of `flag-logo.png` or `power-logo.png` (if they're placeholders).
- Any custom `index.html` you want to use instead of the sample.

I can refine the code or provide additional styling tweaks. Let me know if you need help with deployment or further enhancements! 🇺🇸📊
