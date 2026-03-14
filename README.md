import os import pandas as pd import streamlit as st

FILE_NAME = 'customers_data_gui.csv' COLUMNS = ['رقم_العميل', 'الاسم', 'الرقم_الوطني', 'العنوان']

st.set_page_config(page_title='نظام تحديث بيانات العملاء', page_icon='🧾', layout='centered')

def initialize_db(): if not os.path.exists(FILE_NAME): df = pd.DataFrame(columns=COLUMNS) df.to_csv(FILE_NAME, index=False, encoding='utf-16')

def load_data(): initialize_db() try: df = pd.read_csv(FILE_NAME, encoding='utf-16') except Exception: df = pd.DataFrame(columns=COLUMNS) for col in COLUMNS: if col not in df.columns: df[col] = '' df = df[COLUMNS] df['رقم_العميل'] = df['رقم_العميل'].astype(str) df = df.fillna('') return df

def save_dataframe(df): df.to_csv(FILE_NAME, index=False, encoding='utf-16')

st.title('نظام تحديث بيانات العملاء') st.caption('إضافة أو تحديث بيانات المشتركين بسهولة')

df = load_data()

with st.form('customer_form', clear_on_submit=True): cust_id = st.text_input('رقم العميل (المشترك)') name = st.text_input('الاسم الكامل') national_id = st.text_input('الرقم الوطني') address = st.text_input('العنوان')

submitted = st.form_submit_button('حفظ / تحديث البيانات')

if submitted:
    cust_id = cust_id.strip()
    name = name.strip()
    national_id = national_id.strip()
    address = address.strip()

    if not (cust_id and name and national_id):
        st.warning('يرجى تعبئة الحقول الأساسية: رقم العميل، الاسم، الرقم الوطني.')
    else:
        if cust_id in df['رقم_العميل'].values:
            df.loc[df['رقم_العميل'] == cust_id, ['الاسم', 'الرقم_الوطني', 'العنوان']] = [name, national_id, address]
            save_dataframe(df)
            st.success(f'تم تحديث بيانات العميل رقم {cust_id}')
        else:
            new_row = {
                'رقم_العميل': cust_id,
                'الاسم': name,
                'الرقم_الوطني': national_id,
                'العنوان': address,
            }
            df = pd.concat([df, pd.DataFrame([new_row])], ignore_index=True)
            save_dataframe(df)
            st.success('تم إضافة العميل بنجاح')

st.divider() st.subheader('سجل العملاء')

current_df = load_data() if current_df.empty: st.info('لا توجد بيانات محفوظة حتى الآن.') else: st.dataframe(current_df, use_container_width=True, hide_index=True)

csv_data = current_df.to_csv(index=False, encoding='utf-8-sig').encode('utf-8-sig')
st.download_button(
    label='تحميل البيانات CSV',
    data=csv_data,
    file_name='customers_data_export.csv',
    mime='text/csv'
)

st.divider() st.markdown('طريقة التشغيل محليًا:') st.code('pip install streamlit pandas\nstreamlit run customers_app_streamlit.py', language='bash')
