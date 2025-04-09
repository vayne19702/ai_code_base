# ai_code_base
import pandas as pd
import lightgbm as lgb
from sklearn.preprocessing import LabelEncoder
import numpy as np

# 读取数据
data = pd.read_csv('your_data.csv')  # 请将 'your_data.csv' 替换为你的实际文件名

# 对日期和用户ID进行编码
date_encoder = LabelEncoder()
data['Date_encoded'] = date_encoder.fit_transform(data['Date'])

user_encoder = LabelEncoder()
data['VISITOR_ID_encoded'] = user_encoder.fit_transform(data['VISITOR_ID'])

# 对菜单进行编码
menu_encoder = LabelEncoder()
data['CLICKPATH_encoded'] = menu_encoder.fit_transform(data['CLICKPATH'])

# 准备特征和目标变量
X = data[['Date_encoded', 'VISITOR_ID_encoded']]
y = data['CLICKPATH_encoded']

# 划分训练集和测试集（这里简单地进行划分，实际应用中可以使用更复杂的策略）
from sklearn.model_selection import train_test_split
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, random_state=42)

# 创建LightGBM数据集
lgb_train = lgb.Dataset(X_train, y_train)
lgb_test = lgb.Dataset(X_test, y_test, reference=lgb_train)

# 设置LightGBM参数
params = {
    'boosting_type': 'gbdt',
    'objective':'multiclass',
    'num_class': len(menu_encoder.classes_),
   'metric':'multi_logloss',
    'learning_rate': 0.1,
   'verbose': 0
}

# 训练模型
num_round = 100
model = lgb.train(
    params,
    lgb_train,
    num_round,
    valid_sets=[lgb_train, lgb_test],
    early_stopping_rounds=10
)

# 预测函数
def predict_top3(user_id, date):
    user_id_encoded = user_encoder.transform([user_id])[0]
    date_encoded = date_encoder.transform([date])[0]
    input_data = np.array([[date_encoded, user_id_encoded]])
    predictions = model.predict(input_data)[0]
    top3_indices = np.argsort(predictions)[::-1][:3]
    top3_menus = menu_encoder.inverse_transform(top3_indices)
    top3_scores = predictions[top3_indices]
    return top3_menus, top3_scores

# 示例使用
user_id_example = 'u1'
date_example = '2025-03-05'
top3_menus, top3_scores = predict_top3(user_id_example, date_example)
print(f"用户 {user_id_example} 在 {date_example} 最想访问的Top3菜单及其得分:")
for menu, score in zip(top3_menus, top3_scores):
    print(f"菜单: {menu}, 得分: {score}")