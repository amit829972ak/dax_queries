import streamlit as st
import pandas as pd
import google.generativeai as genai
import os

# Configure Streamlit page
st.set_page_config(page_title="DAX Query Generator", page_icon="📊", layout="wide")

st.title("📊 AI-Powered DAX Query Generator")
st.markdown("""
This app uses **Google Gemini** to generate Power BI DAX queries based on your uploaded data and natural language prompts.
""")

# Sidebar for configuration
with st.sidebar:
    st.header("Configuration")
    
    # API Key handling
    api_key_input = st.text_input("Enter Gemini API Key", type="password", value=os.environ.get("GEMINI_API_KEY", ""))
    if api_key_input:
        api_key = api_key_input
        # Also update session state or environment if needed, but for now just use the variable
    else:
        api_key = os.environ.get("GEMINI_API_KEY")
    
    # Model Selection
    # Defaulting to 2.5-flash as requested by the user.
    model_option = st.selectbox(
        "Select Gemini Model",
        options=["gemini-2.0-flash", "gemini-2.5-flash", "gemini-1.5-pro", "gemini-1.5-flash", "gemini-1.0-pro"],
        index=1
    )
    
    st.markdown("---")
    st.markdown("### Instructions")
    st.markdown("1. Upload your dataset (CSV, TSV, or Excel)")
    st.markdown("2. Preview your data")
    st.markdown("3. Describe the metric you want to calculate")
    st.markdown("4. Get the DAX query!")

# Function to get Gemini response
def get_gemini_response(prompt, api_key, model_name):
    try:
        genai.configure(api_key=api_key)
        model = genai.GenerativeModel(model_name)
        response = model.generate_content(prompt)
        return response.text
    except Exception as e:
        return f"Error: {str(e)}"

# Main app logic
if not api_key:
    st.warning("⚠️ Please enter your Google Gemini API Key in the sidebar to proceed.")
else:
    # Use cache_data to speed up file processing
    @st.cache_data(show_spinner=False)
    def load_data(file):
        file_ext = file.name.split('.')[-1].lower()
        if file_ext == 'csv':
            return pd.read_csv(file)
        elif file_ext == 'tsv':
            return pd.read_csv(file, sep='\t')
        elif file_ext in ['xlsx', 'xls']:
            return pd.read_excel(file)
        return None

    uploaded_file = st.file_uploader("Upload your file (CSV, TSV, or Excel)", type=['csv', 'tsv', 'xlsx', 'xls'])

    if uploaded_file is not None:
        try:
            with st.spinner("Loading file..."):
                df = load_data(uploaded_file)
            
            if df is not None:
                # Display Data Preview
                st.subheader("Data Preview")
                st.dataframe(df.head())

                # Extract schema info
                dtypes = df.dtypes.to_dict()
                schema_info = "\n".join([f"- {col} ({dtype})" for col, dtype in dtypes.items()])

                st.subheader("Generate DAX Query")
                user_prompt = st.text_area("Describe what you want to calculate (e.g., 'Total Sales for previous year')", height=100)
                
                if st.button("Generate DAX", type="primary"):
                    if user_prompt:
                        with st.spinner(f"Generating DAX query using {model_option}..."):
                            # Construct the prompt for Gemini
                            prompt = f"""
                            You are an expert Power BI DAX developer. 
                            I have a table named 'Table1' with the following columns and data types:
                            {schema_info}

                            User Request: {user_prompt}

                            Please provide the correct DAX formula to achieve this. 
                            - Use 'Table1' as the table name.
                            - Explain the logic briefly.
                            - Wrap the DAX code in a code block.
                            """
                            
                            response = get_gemini_response(prompt, api_key, model_option)
                            st.markdown("### Generated DAX Query")
                            st.markdown(response)
                    else:
                        st.warning("Please enter a prompt.")
            else:
                st.error("Unsupported file format.")
        
        except Exception as e:
            st.error(f"Error reading file: {e}")
