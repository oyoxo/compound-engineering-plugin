---
name: streamlit-patterns
description: Best practices and patterns for building production Streamlit applications. Use when creating new Streamlit apps, refactoring existing apps, or solving common Streamlit challenges.
---

<objective>
Provide comprehensive guidance for building maintainable, performant Streamlit applications following established patterns and avoiding common pitfalls.
</objective>

<quick_start>
When building a Streamlit app:
1. Initialize session state centrally at app start
2. Separate business logic from UI (no `st.*` in services)
3. Use caching strategically (`@st.cache_data`, `@st.cache_resource`)
4. Structure layout: inputs → action → results
</quick_start>

<essential_patterns>

## Session State Initialization

ALWAYS initialize all session state keys at the start of your app:

```python
# app.py - at the very top, after imports
def init_session_state():
    """Initialize all session state variables with defaults."""
    defaults = {
        # User inputs
        "user_query": "",
        "selected_option": None,
        
        # Processing state
        "is_processing": False,
        "results": None,
        "error": None,
        
        # UI state
        "current_page": "home",
        "sidebar_expanded": True,
    }
    for key, default in defaults.items():
        if key not in st.session_state:
            st.session_state[key] = default

# Call immediately
init_session_state()
```

## The Rerun Mental Model

Every widget interaction triggers a full script rerun. Design for this:

```python
# BAD: Computation runs on every rerun
data = load_expensive_data()  # Runs every time!
processed = heavy_computation(data)  # And this too!

# GOOD: Cache expensive operations
@st.cache_data
def load_expensive_data():
    return pd.read_csv("large_file.csv")

@st.cache_data  
def heavy_computation(data):
    return data.groupby("category").agg(...)

data = load_expensive_data()  # Cached
processed = heavy_computation(data)  # Cached
```

## Caching Strategy

| Decorator | Use For | Example |
|-----------|---------|---------|
| `@st.cache_data` | Data that can be serialized | DataFrames, API responses, computed results |
| `@st.cache_resource` | Global resources | DB connections, ML models, API clients |

```python
@st.cache_resource
def get_database_connection():
    """Singleton connection - lives across reruns."""
    return create_engine(DATABASE_URL)

@st.cache_data(ttl=3600)  # Expire after 1 hour
def fetch_weather_data(city: str) -> dict:
    """Cached API call with TTL."""
    response = requests.get(f"https://api.weather.com/{city}")
    return response.json()

@st.cache_data(show_spinner="Loading model...")
def load_ml_model(model_path: str):
    """With custom spinner message."""
    return joblib.load(model_path)
```

## Service Layer Separation

Keep business logic streamlit-free for testability:

```python
# services/analyzer.py - NO streamlit imports!
from dataclasses import dataclass

@dataclass
class AnalysisResult:
    summary: str
    metrics: dict[str, float]
    recommendations: list[str]

def analyze_data(df: pd.DataFrame) -> AnalysisResult:
    """Pure function - easy to test."""
    return AnalysisResult(
        summary=generate_summary(df),
        metrics=calculate_metrics(df),
        recommendations=get_recommendations(df),
    )

# app.py - streamlit integration
from services.analyzer import analyze_data, AnalysisResult

@st.cache_data
def get_analysis(df: pd.DataFrame) -> AnalysisResult:
    return analyze_data(df)

# Now you can test analyze_data() without Streamlit
```

## Layout Patterns

### Input → Action → Result

```python
st.header("Data Analyzer")

# === INPUT SECTION ===
with st.sidebar:
    uploaded_file = st.file_uploader("Upload CSV", type="csv")
    analysis_type = st.selectbox("Analysis Type", ["Summary", "Detailed"])
    
# === ACTION ===
if st.button("Analyze", disabled=uploaded_file is None):
    with st.spinner("Analyzing..."):
        st.session_state.results = run_analysis(uploaded_file, analysis_type)

# === RESULTS ===
if st.session_state.results:
    display_results(st.session_state.results)
```

### Multi-Column Layout

```python
col1, col2, col3 = st.columns([2, 1, 1])  # Relative widths

with col1:
    st.subheader("Main Content")
    st.dataframe(df)

with col2:
    st.subheader("Metrics")
    st.metric("Total", len(df))
    
with col3:
    st.subheader("Actions")
    if st.button("Export"):
        export_data(df)
```

### Tabs for Related Content

```python
tab1, tab2, tab3 = st.tabs(["📊 Data", "📈 Charts", "📝 Report"])

with tab1:
    st.dataframe(df)
    
with tab2:
    st.plotly_chart(create_chart(df))
    
with tab3:
    st.markdown(generate_report(df))
```

## Callback Pattern

Use callbacks for complex state updates:

```python
def on_file_upload():
    """Process file when uploaded."""
    file = st.session_state.uploaded_file
    if file:
        st.session_state.data = pd.read_csv(file)
        st.session_state.filename = file.name
        st.session_state.analysis_ready = True

st.file_uploader(
    "Upload CSV",
    type="csv",
    key="uploaded_file",
    on_change=on_file_upload,
)
```

## Error Handling

```python
def safe_operation():
    try:
        with st.spinner("Processing..."):
            result = risky_operation()
            st.success("Done!")
            return result
    except ValidationError as e:
        st.error(f"Invalid input: {e}")
    except APIError as e:
        st.error(f"API error: {e}")
        st.info("Please try again later.")
    except Exception as e:
        st.error("An unexpected error occurred.")
        st.exception(e)  # Shows full traceback in dev
    return None
```

## Form Pattern (Batch Submissions)

When you want to submit multiple inputs at once:

```python
with st.form("settings_form"):
    st.subheader("Configuration")
    
    name = st.text_input("Name")
    age = st.number_input("Age", min_value=0, max_value=150)
    preferences = st.multiselect("Preferences", ["A", "B", "C"])
    
    # Form submission - only triggers ONE rerun
    submitted = st.form_submit_button("Save Settings")
    
    if submitted:
        save_settings(name, age, preferences)
        st.success("Settings saved!")
```

</essential_patterns>

<project_structure>
## Recommended File Structure

```
my_streamlit_app/
├── app.py                    # Main entry point
├── config.py                 # Settings and constants
├── requirements.txt
│
├── pages/                    # Multi-page app (auto-discovered)
│   ├── 1_📊_Dashboard.py
│   ├── 2_🔍_Analysis.py
│   └── 3_⚙️_Settings.py
│
├── components/               # Reusable UI components
│   ├── __init__.py
│   ├── sidebar.py
│   ├── charts.py
│   └── data_display.py
│
├── services/                 # Business logic (NO streamlit!)
│   ├── __init__.py
│   ├── data_loader.py
│   ├── analyzer.py
│   └── exporter.py
│
├── models/                   # Pydantic models / dataclasses
│   ├── __init__.py
│   └── schemas.py
│
├── utils/                    # Helpers
│   ├── __init__.py
│   └── formatters.py
│
└── .streamlit/
    ├── config.toml           # UI configuration
    └── secrets.toml          # API keys (gitignored!)
```

## Multi-Page App Entry Point

```python
# app.py
import streamlit as st
from components.sidebar import render_sidebar
from config import APP_TITLE

st.set_page_config(
    page_title=APP_TITLE,
    page_icon="🚀",
    layout="wide",
    initial_sidebar_state="expanded",
)

def init_session_state():
    # ... initialization
    pass

def main():
    init_session_state()
    render_sidebar()
    
    st.title(APP_TITLE)
    st.markdown("Welcome! Select a page from the sidebar.")

if __name__ == "__main__":
    main()
```

</project_structure>

<performance_tips>
## Performance Optimization

### 1. Dataframe Display
```python
# BAD: Loading huge dataframe
st.dataframe(huge_df)  # Browser struggles with 100k+ rows

# GOOD: Pagination
page_size = 100
page = st.number_input("Page", min_value=1, max_value=len(df)//page_size + 1)
start = (page - 1) * page_size
st.dataframe(df.iloc[start:start + page_size])
```

### 2. Chart Caching
```python
@st.cache_data
def create_chart(df: pd.DataFrame) -> go.Figure:
    """Cache the entire figure object."""
    fig = px.scatter(df, x="x", y="y")
    return fig

st.plotly_chart(create_chart(df), use_container_width=True)
```

### 3. Lazy Loading
```python
if st.checkbox("Show advanced options"):
    # Only compute when needed
    advanced_data = compute_advanced_metrics(df)
    st.write(advanced_data)
```

### 4. Fragment Decorator (Streamlit 1.33+)
```python
@st.fragment
def chat_input_section():
    """Only this section reruns on interaction."""
    user_input = st.text_input("Message")
    if st.button("Send"):
        process_message(user_input)

# Main app doesn't rerun when fragment interacts
st.header("My App")
expensive_computation()
chat_input_section()  # Isolated reruns
```

</performance_tips>

<secrets_config>
## Secrets & Configuration

### .streamlit/secrets.toml
```toml
# Never commit this file!
OPENAI_API_KEY = "sk-..."
DATABASE_URL = "postgresql://..."

[aws]
access_key = "..."
secret_key = "..."
```

### Usage in Code
```python
import streamlit as st

# Access secrets
api_key = st.secrets["OPENAI_API_KEY"]
aws_key = st.secrets["aws"]["access_key"]

# With fallback for local dev
api_key = st.secrets.get("OPENAI_API_KEY", os.getenv("OPENAI_API_KEY"))
```

### config.py for Non-Secrets
```python
from dataclasses import dataclass

@dataclass(frozen=True)
class Config:
    APP_TITLE: str = "My Data App"
    DEFAULT_PAGE_SIZE: int = 50
    MAX_UPLOAD_SIZE_MB: int = 200
    SUPPORTED_FILE_TYPES: tuple = ("csv", "xlsx", "json")

config = Config()
```

</secrets_config>

<success_criteria>
A well-built Streamlit app:
- [ ] Has centralized session state initialization
- [ ] Uses caching appropriately for expensive operations
- [ ] Separates business logic from UI (testable services)
- [ ] Has clear visual hierarchy: inputs → action → results
- [ ] Handles errors gracefully with user feedback
- [ ] Uses secrets.toml for sensitive config
- [ ] Follows multi-page structure for complex apps
- [ ] Streams long operations with progress indicators
</success_criteria>
