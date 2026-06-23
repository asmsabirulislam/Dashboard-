# ⚙️ Advanced Customization Guide

Professional customization and integration examples for the Enterprise Dashboard.

---

## 📋 Table of Contents
1. [Color Customization](#-color-customization)
2. [Data Integration](#-data-integration)
3. [Adding Charts](#-adding-charts)
4. [Custom Metrics](#-custom-metrics)
5. [Advanced Filters](#-advanced-filters)
6. [Performance Optimization](#-performance-optimization)
7. [Security & Authentication](#-security--authentication)
8. [Deployment Configuration](#-deployment-configuration)

---

## 🎨 Color Customization

### Change Theme Colors

Edit the `THEMES` dictionary to customize colors:

```python
THEMES = {
    "dark": {
        "bg_primary": "#0a0e27",      # Main background
        "bg_secondary": "#1a1f3a",    # Sidebar background
        "text_primary": "#e2eaf3",    # Main text
        "accent_primary": "#00d4ff",  # Primary accent (Cyan)
        "accent_secondary": "#00ff88",# Secondary accent (Green)
        "accent_danger": "#ff4757",   # Error/reject color
        "accent_success": "#2ed573",  # Success color
        "accent_warning": "#ffa502",  # Warning color
        "gradient_1": "linear-gradient(135deg, #667eea 0%, #764ba2 100%)",
    }
}
```

### Example: Corporate Blue Theme

```python
"corporate_blue": {
    "bg_primary": "#1a2744",
    "bg_secondary": "#243456",
    "text_primary": "#e8eef5",
    "accent_primary": "#0066cc",
    "accent_secondary": "#0099ff",
    "accent_success": "#00aa44",
    "accent_danger": "#cc0000",
    "gradient_1": "linear-gradient(135deg, #0066cc 0%, #0099ff 100%)",
}
```

### Example: Green Sustainability Theme

```python
"sustainability": {
    "bg_primary": "#0d1b0f",
    "bg_secondary": "#152617",
    "text_primary": "#e8f5e9",
    "accent_primary": "#00aa44",
    "accent_secondary": "#66dd66",
    "accent_success": "#2ed573",
    "accent_danger": "#ff6b6b",
    "gradient_1": "linear-gradient(135deg, #00aa44 0%, #66dd66 100%)",
}
```

---

## 📊 Data Integration

### Loading from CSV

```python
@st.cache_data
def load_csv_data():
    return pd.read_csv('financial_data.csv')

# Usage
data = load_csv_data()
```

### Loading from Excel

```python
@st.cache_data
def load_excel_data(sheet_name='Data'):
    return pd.read_excel('financial_data.xlsx', sheet_name=sheet_name)

data = load_excel_data()
```

### Loading from SQL Database

```python
import sqlite3
from sqlalchemy import create_engine

@st.cache_data
def load_sql_data():
    engine = create_engine('sqlite:///financial.db')
    query = "SELECT * FROM transactions WHERE date >= DATE('now', '-1 year')"
    data = pd.read_sql(query, engine)
    return data

data = load_sql_data()
```

### PostgreSQL Integration

```python
@st.cache_data
def load_postgresql_data():
    engine = create_engine(
        'postgresql://user:password@localhost:5432/financial_db'
    )
    query = """
    SELECT * FROM transactions 
    WHERE date >= NOW() - INTERVAL '1 year'
    ORDER BY date DESC
    """
    data = pd.read_sql(query, engine)
    return data
```

### Real-Time Data Streaming

```python
@st.cache_resource
def get_live_data_connection():
    # Your connection setup
    return connection

def load_live_data():
    # No cache - always fresh
    connection = get_live_data_connection()
    data = pd.read_sql("SELECT * FROM transactions", connection)
    return data

# In your app
if st.button("🔄 Refresh Live Data"):
    st.session_state.data = load_live_data()
    st.rerun()
```

---

## 📈 Adding Charts

### Add a New Chart Type: Histogram

```python
# In appropriate tab section
col1, col2 = st.columns(2)

with col1:
    st.markdown("#### Invoice Distribution (Histogram)")
    
    fig_histogram = go.Figure()
    fig_histogram.add_trace(go.Histogram(
        x=filtered_data['Invoice Value'],
        nbinsx=50,
        marker=dict(color=colors['accent_primary']),
        name='Invoice Distribution'
    ))
    
    fig_histogram.update_layout(
        **get_plotly_config(),
        title="Invoice Amount Distribution",
        xaxis_title="Invoice Value ($)",
        yaxis_title="Frequency",
        height=400,
        showlegend=False
    )
    
    st.plotly_chart(fig_histogram, use_container_width=True)
```

### Add Waterfall Chart

```python
# Show profit flow
data_waterfall = pd.DataFrame({
    'Category': ['Revenue', 'Costs', 'Taxes', 'Net Profit'],
    'Amount': [100000, -30000, -15000, 55000],
    'Type': ['absolute', 'relative', 'relative', 'total']
})

fig_waterfall = go.Figure(go.Waterfall(
    x=data_waterfall['Category'],
    y=data_waterfall['Amount'],
    connector={"line": {"color": colors['accent_secondary']}},
    decreasing={"marker": {"color": colors['accent_danger']}},
    increasing={"marker": {"color": colors['accent_success']}}
))

fig_waterfall.update_layout(
    **get_plotly_config(),
    title="Financial Waterfall",
    height=400
)

st.plotly_chart(fig_waterfall, use_container_width=True)
```

### Add Gauge Chart

```python
# KPI gauge
def create_gauge(value, title, max_value=100):
    fig = go.Figure(go.Indicator(
        mode="gauge+number+delta",
        value=value,
        title={'text': title},
        delta={'reference': max_value * 0.8},
        gauge={
            'axis': {'range': [0, max_value]},
            'bar': {'color': colors['accent_primary']},
            'steps': [
                {'range': [0, max_value * 0.5], 'color': colors['bg_tertiary']},
                {'range': [max_value * 0.5, max_value], 'color': colors['bg_secondary']}
            ],
            'threshold': {
                'line': {'color': colors['accent_danger'], 'width': 4},
                'thickness': 0.75,
                'value': max_value * 0.9
            }
        }
    ))
    
    fig.update_layout(
        height=300,
        paper_bgcolor=colors['card_bg'],
        font=dict(color=colors['text_primary'])
    )
    
    return fig

# Usage
acceptance_rate = 85
fig = create_gauge(acceptance_rate, "Acceptance Rate (%)", 100)
st.plotly_chart(fig, use_container_width=True)
```

### Add Funnel Chart

```python
# Sales funnel
funnel_data = pd.DataFrame({
    'Stage': ['Leads', 'Interested', 'Quoted', 'Won'],
    'Count': [1000, 500, 250, 100]
})

fig_funnel = go.Figure(go.Funnel(
    y=funnel_data['Stage'],
    x=funnel_data['Count'],
    marker=dict(color=COLORS_PALETTE)
))

fig_funnel.update_layout(
    **get_plotly_config(),
    title="Sales Funnel",
    height=400
)

st.plotly_chart(fig_funnel, use_container_width=True)
```

---

## 🎯 Custom Metrics

### Add New KPI Card

```python
def custom_metric_card(
    title: str,
    value: str,
    subtitle: str = "",
    icon: str = "📊",
    trend: float = None,
    status: str = "info"
):
    """
    Create a custom metric card
    status: 'success', 'warning', 'danger', 'info'
    """
    status_colors = {
        'success': colors['accent_success'],
        'warning': colors['accent_warning'],
        'danger': colors['accent_danger'],
        'info': colors['accent_primary']
    }
    
    color = status_colors.get(status, colors['accent_primary'])
    
    trend_html = ""
    if trend is not None:
        trend_emoji = "📈" if trend > 0 else "📉" if trend < 0 else "➡️"
        trend_color = "green" if trend > 0 else "red" if trend < 0 else "gray"
        trend_html = f'''
        <div style="color: {trend_color}; font-size: 12px; margin-top: 8px;">
            {trend_emoji} {abs(trend):.1f}%
        </div>
        '''
    
    html = f'''
    <div class="metric-card" style="border-left: 4px solid {color};">
        <div style="color: {colors['text_secondary']}; font-size: 12px; font-weight: 700; margin-bottom: 8px;">
            {icon} {title}
        </div>
        <div style="font-size: 32px; font-weight: 700; color: {colors['text_primary']};">
            {value}
        </div>
        {f'<div style="font-size: 12px; color: {colors["text_secondary"]}; margin-top: 4px;">{subtitle}</div>' if subtitle else ''}
        {trend_html}
    </div>
    '''
    
    st.markdown(html, unsafe_allow_html=True)

# Usage
custom_metric_card(
    title="Market Share",
    value="24.5%",
    subtitle="Top 3 competitor",
    icon="📈",
    trend=5.2,
    status="success"
)
```

### Calculate Advanced Metrics

```python
def calculate_metrics(data: pd.DataFrame) -> dict:
    """Calculate advanced financial metrics"""
    
    total_revenue = data['Invoice Value'].sum()
    transaction_count = len(data)
    avg_transaction = data['Invoice Value'].mean()
    median_transaction = data['Invoice Value'].median()
    std_transaction = data['Invoice Value'].std()
    
    # Payment metrics
    payment_received = len(data[data['Payment Status'] == 'Received'])
    payment_recovery_rate = (payment_received / transaction_count * 100) if transaction_count > 0 else 0
    
    # Acceptance metrics
    accepted = len(data[data['Status'] == 'Accepted'])
    acceptance_rate = (accepted / transaction_count * 100) if transaction_count > 0 else 0
    
    # Growth metrics
    data['Month'] = pd.to_datetime(data['Date']).dt.to_period('M')
    monthly_revenue = data.groupby('Month')['Invoice Value'].sum()
    if len(monthly_revenue) > 1:
        current_month = monthly_revenue.iloc[-1]
        previous_month = monthly_revenue.iloc[-2]
        growth_rate = ((current_month - previous_month) / previous_month * 100) if previous_month > 0 else 0
    else:
        growth_rate = 0
    
    return {
        'total_revenue': total_revenue,
        'transaction_count': transaction_count,
        'avg_transaction': avg_transaction,
        'median_transaction': median_transaction,
        'std_transaction': std_transaction,
        'payment_recovery_rate': payment_recovery_rate,
        'acceptance_rate': acceptance_rate,
        'growth_rate': growth_rate
    }

# Usage
metrics = calculate_metrics(filtered_data)
print(f"Total Revenue: ${metrics['total_revenue']:,.2f}")
print(f"Acceptance Rate: {metrics['acceptance_rate']:.1f}%")
```

---

## 🔍 Advanced Filters

### Add Multi-Select with Search

```python
def advanced_multi_select(label: str, options: list, key: str = None):
    """Multi-select with search and select-all"""
    col1, col2 = st.columns([4, 1])
    
    with col1:
        selected = st.multiselect(label, options, key=key)
    
    with col2:
        if st.button("All", key=f"all_{key}"):
            st.session_state[key] = options
            st.rerun()
    
    return selected

# Usage
selected_items = advanced_multi_select(
    "Select Items",
    options=["Item 1", "Item 2", "Item 3"],
    key="items"
)
```

### Add Date Range Presets

```python
st.markdown("### Quick Date Ranges")

col1, col2, col3, col4, col5 = st.columns(5)

with col1:
    if st.button("📅 Last 7 Days"):
        end_date = datetime.now().date()
        start_date = end_date - timedelta(days=7)
        st.session_state.date_range = (start_date, end_date)
        st.rerun()

with col2:
    if st.button("📅 Last 30 Days"):
        end_date = datetime.now().date()
        start_date = end_date - timedelta(days=30)
        st.session_state.date_range = (start_date, end_date)
        st.rerun()

with col3:
    if st.button("📅 Last Quarter"):
        end_date = datetime.now().date()
        start_date = end_date - timedelta(days=90)
        st.session_state.date_range = (start_date, end_date)
        st.rerun()

with col4:
    if st.button("📅 This Year"):
        end_date = datetime.now().date()
        start_date = end_date.replace(month=1, day=1)
        st.session_state.date_range = (start_date, end_date)
        st.rerun()

with col5:
    if st.button("📅 YTD"):
        now = datetime.now().date()
        ytd_start = now.replace(month=1, day=1)
        st.session_state.date_range = (ytd_start, now)
        st.rerun()
```

---

## ⚡ Performance Optimization

### Caching Strategy

```python
# Cache expensive computations
@st.cache_data(ttl=3600)  # Cache for 1 hour
def expensive_calculation(data: pd.DataFrame):
    # Complex analysis
    return result

# Cache database connections
@st.cache_resource
def get_database_connection():
    return database.connect()

# Manual cache control
if st.button("Clear Cache"):
    st.cache_data.clear()
    st.cache_resource.clear()
    st.success("Cache cleared!")
```

### Lazy Loading Large Data

```python
def load_data_in_chunks(file_path, chunksize=10000):
    """Load data in chunks for memory efficiency"""
    chunks = []
    for chunk in pd.read_csv(file_path, chunksize=chunksize):
        # Process chunk if needed
        chunks.append(chunk)
    return pd.concat(chunks, ignore_index=True)

# For very large files, use generators
def read_large_data(file_path):
    for chunk in pd.read_csv(file_path, chunksize=50000):
        yield chunk
```

### Profile Performance

```python
import time
import cProfile
import pstats
from io import StringIO

def profile_function(func):
    """Decorator to profile function execution"""
    def wrapper(*args, **kwargs):
        pr = cProfile.Profile()
        pr.enable()
        
        start = time.time()
        result = func(*args, **kwargs)
        elapsed = time.time() - start
        
        pr.disable()
        s = StringIO()
        ps = pstats.Stats(pr, stream=s).sort_stats('cumulative')
        ps.print_stats(10)
        
        st.write(f"Execution time: {elapsed:.2f}s")
        st.code(s.getvalue())
        
        return result
    return wrapper

@profile_function
def slow_operation(data):
    # Your code here
    return processed_data
```

---

## 🔐 Security & Authentication

### Basic Authentication

```python
import streamlit_authenticator as stauth

names = ["John Smith", "Jane Doe"]
usernames = ["jsmith", "jdoe"]
passwords = ["password123", "password456"]  # Hash these!

authenticator = stauth.Authenticate(
    names,
    usernames,
    passwords,
    "dashboard_key",
    "signature_key",
    cookie_expiry_days=30
)

name, authentication_status, username = authenticator.login(
    "Login",
    "main"
)

if authentication_status:
    # Show dashboard
    st.write(f"Welcome {name}!")
    
    if authenticator.logout("Logout", "sidebar"):
        st.success("Logged out successfully!")
        st.stop()
        
    # Your dashboard code here
    
elif authentication_status is False:
    st.error("Username or password is incorrect")
elif authentication_status is None:
    st.warning("Please enter your username and password")
```

### Environment-Based Secrets

```python
import os
from dotenv import load_dotenv

load_dotenv()

DATABASE_URL = os.getenv("DATABASE_URL")
API_KEY = os.getenv("API_KEY")

# .env file (never commit to git)
# DATABASE_URL=postgresql://user:pass@localhost/db
# API_KEY=sk_live_xxxxx
```

---

## 🚀 Deployment Configuration

### Streamlit Secrets

Create `.streamlit/secrets.toml`:

```toml
[database]
url = "postgresql://user:password@localhost/db"
port = 5432

[api]
key = "your-api-key"
endpoint = "https://api.example.com"

[email]
username = "your-email@gmail.com"
password = "your-app-password"
```

Access in code:
```python
db_url = st.secrets["database"]["url"]
api_key = st.secrets["api"]["key"]
```

### Docker Configuration

Create `Dockerfile`:

```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8501

CMD ["streamlit", "run", "powerbi_enterprise_dashboard.py", "--server.port=8501"]
```

Create `.dockerignore`:

```
.git
.gitignore
__pycache__
*.pyc
.pytest_cache
.env
```

Run:
```bash
docker build -t powerbi-dashboard .
docker run -p 8501:8501 powerbi-dashboard
```

### Heroku Deployment

Create `Procfile`:

```
web: streamlit run powerbi_enterprise_dashboard.py --logger.level=error
```

Create `runtime.txt`:

```
python-3.11.0
```

Deploy:

```bash
git push heroku main
```

### AWS Lambda (Serverless)

```python
from mangum import Mangum
from streamlit.web.cli import main

app = Mangum(main)
```

---

## 📊 Custom Report Generator

```python
def generate_custom_report(
    filtered_data: pd.DataFrame,
    title: str,
    sections: list
) -> str:
    """Generate custom report in markdown"""
    
    report = f"""
# {title}

**Generated**: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}

## Executive Summary

- Total Records: {len(filtered_data):,}
- Total Value: ${filtered_data['Invoice Value'].sum():,.2f}
- Acceptance Rate: {(len(filtered_data[filtered_data['Status']=='Accepted'])/len(filtered_data)*100):.1f}%
- Payment Recovery: {(len(filtered_data[filtered_data['Payment Status']=='Received'])/len(filtered_data)*100):.1f}%

"""
    
    for section in sections:
        report += f"\n## {section['title']}\n\n"
        report += section['content'] + "\n"
    
    return report

# Usage
sections = [
    {
        'title': 'Top Performers',
        'content': "Best Bank: Global Bank"
    },
    {
        'title': 'Key Insights',
        'content': "Revenue growth of 25% YoY"
    }
]

report = generate_custom_report(filtered_data, "Q4 Report", sections)
st.markdown(report)

st.download_button("Download Report", report, "report.md", "text/markdown")
```

---

## 🎓 Best Practices

### Code Organization

```
powerbi-dashboard/
├── powerbi_enterprise_dashboard.py  # Main app
├── requirements.txt                  # Dependencies
├── config.py                        # Configuration
├── utils/
│   ├── __init__.py
│   ├── data_loader.py              # Data loading
│   ├── charts.py                   # Chart functions
│   ├── filters.py                  # Filter logic
│   └── metrics.py                  # Metric calculations
├── pages/                          # Multi-page apps
│   ├── executive.py
│   ├── analysis.py
│   └── reports.py
└── .env                            # Secrets (gitignore)
```

### Configuration Best Practices

```python
# config.py
import os

class Config:
    # Theme
    THEME = os.getenv("THEME", "dark")
    
    # Data
    DATA_SOURCE = os.getenv("DATA_SOURCE", "sample")
    DATABASE_URL = os.getenv("DATABASE_URL")
    
    # Performance
    CACHE_TTL = int(os.getenv("CACHE_TTL", "3600"))
    CHUNK_SIZE = int(os.getenv("CHUNK_SIZE", "10000"))
    
    # Security
    REQUIRE_AUTH = os.getenv("REQUIRE_AUTH", "false").lower() == "true"
    
class DevelopmentConfig(Config):
    DEBUG = True

class ProductionConfig(Config):
    DEBUG = False
    CACHE_TTL = 7200
```

---

## 🔗 Integration Examples

### Integration with Power BI

```python
# Export to Power BI via API
def export_to_power_bi(data: pd.DataFrame, dataset_id: str):
    import requests
    
    # Convert to JSON
    payload = data.to_json()
    
    # Push to Power BI
    response = requests.post(
        f"https://api.powerbi.com/v1.0/myorg/datasets/{dataset_id}/rows",
        headers={"Authorization": f"Bearer {access_token}"},
        json=payload
    )
    
    return response.status_code == 200
```

### Integration with Slack

```python
def send_report_to_slack(message: str, webhook_url: str):
    import requests
    
    payload = {
        "text": message,
        "blocks": [
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": message
                }
            }
        ]
    }
    
    response = requests.post(webhook_url, json=payload)
    return response.status_code == 200

# Usage
if st.button("📤 Share to Slack"):
    summary = "Daily Report Summary: Revenue up 15%, Acceptance 92%"
    send_report_to_slack(summary, st.secrets["slack"]["webhook_url"])
```

---

<div align="center">

## 🎉 You're Ready for Advanced Customization!

Choose what to implement first and start building your custom dashboard.

For questions, refer to the **README.md** or Streamlit documentation.

</div>
