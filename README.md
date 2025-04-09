# ai_code_base
import pandas as pd
import lightgbm as lgb
from sklearn.preprocessing import LabelEncoder
from sklearn.metrics import classification_report

# === 1. 读取 Excel 数据 ===
df = pd.read_excel('your_data.xlsx')

# === 2. 特征工程 ===
df['Date'] = pd.to_datetime(df['Date'])
df['day'] = df['Date'].dt.day
df['month'] = df['Date'].dt.month
df['weekday'] = df['Date'].dt.weekday

# 编码 VISITOR_ID 和 CLICKPATH
user_encoder = LabelEncoder()
menu_encoder = LabelEncoder()
df['VISITOR_ID_enc'] = user_encoder.fit_transform(df['VISITOR_ID'])
df['CLICKPATH_enc'] = menu_encoder.fit_transform(df['CLICKPATH'])

# === 3. 添加历史行为特征 ===
# 计算每个用户的历史菜单访问频率
user_menu_freq = df.groupby(['VISITOR_ID_enc', 'CLICKPATH_enc']).agg({'PageViews': 'sum'}).reset_index()
user_menu_freq['user_menu_frequency'] = user_menu_freq['PageViews'] / user_menu_freq.groupby('VISITOR_ID_enc')['PageViews'].transform('sum')

# === 4. 为每个用户添加过去菜单的偏好（历史特征） ===
df = df.merge(user_menu_freq[['VISITOR_ID_enc', 'CLICKPATH_enc', 'user_menu_frequency']], 
              on=['VISITOR_ID_enc', 'CLICKPATH_enc'], 
              how='left')

# === 5. 准备训练数据 ===
features = ['VISITOR_ID_enc', 'day', 'month', 'weekday', 'user_menu_frequency']
X = df[features]
y = df['CLICKPATH_enc']

# === 6. 训练模型 ===
model = lgb.LGBMClassifier(objective='multiclass',
                           num_class=len(menu_encoder.classes_),
                           random_state=42)
model.fit(X, y)  # 训练时不传递 sample_weight

# === 7. 预测与评估模型 ===
y_pred = model.predict(X)
print(classification_report(y, y_pred, target_names=menu_encoder.classes_))
