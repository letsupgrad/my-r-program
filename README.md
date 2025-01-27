# Load necessary libraries
library(shiny)
library(shinydashboard)
library(mongolite)

# MongoDB connection string (update credentials if needed)
mongo_uri <- "mongodb+srv://sangita:B@production-titan-cluste.ysuki.mongodb.net/master?readPreference=secondaryPreferred&ssl=true"

# UI for the Shiny app
ui <- dashboardPage(
  dashboardHeader(title = "MongoDB Connection Check"),
  dashboardSidebar(
    sidebarMenu(
      menuItem("Connection Status", tabName = "connection", icon = icon("server"))
    )
  ),
  dashboardBody(
    tabItems(
      tabItem(
        tabName = "connection",
        fluidRow(
          valueBoxOutput("uni_status"),
          valueBoxOutput("dsp_status")
        )
      )
    )
  )
)

# Server logic
server <- function(input, output, session) {
  
  # Reactive to check connection for `uni_billboards`
  uni_status <- reactive({
    tryCatch({
      conn <- mongo(collection = "uni_billboards", db = "master", url = mongo_uri)
      conn$count()  # Check if documents exist
      conn$disconnect()  # Disconnect after checking
      "Connected"
    }, error = function(e) {
      "Failed"
    })
  })
  
  # Reactive to check connection for `dsp_line_item_daily_report_inventory`
  dsp_status <- reactive({
    tryCatch({
      conn <- mongo(collection = "dsp_line_item_daily_report_inventory", db = "master", url = mongo_uri)
      conn$count()  # Check if documents exist
      conn$disconnect()  # Disconnect after checking
      "Connected"
    }, error = function(e) {
      "Failed"
    })
  })
  
  # Display connection status for `uni_billboards`
  output$uni_status <- renderValueBox({
    status <- uni_status()
    color <- ifelse(status == "Connected", "green", "red")
    valueBox(status, subtitle = "Connection to 'uni_billboards' Collection", color = color)
  })
  
  # Display connection status for `dsp_line_item_daily_report_inventory`
  output$dsp_status <- renderValueBox({
    status <- dsp_status()
    color <- ifelse(status == "Connected", "green", "red")
    valueBox(status, subtitle = "Connection to 'dsp_line_item_daily_report_inventory' Collection", color = color)
  })
}

# Run the app
shinyApp(ui = ui, server = server)
