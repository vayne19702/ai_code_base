# ai_code_base
import pandas as pd
import numpy as np
from sklearn.preprocessing import LabelEncoder
from sklearn.model_selection import train_test_split
import lightgbm as lgb
from datetime import datetime

# 1. 数据加载与预处理
df = pd.read_excel("user_menu_data.xlsx")

# 转换日期格式
df["Date"] = pd.to_datetime(df["Date"])

# 2. 特征工程
# 编码分类特征
user_encoder = LabelEncoder()
menu_encoder = LabelEncoder()
df["user_id"] = user_encoder.fit_transform(df["VISITOR_ID"])
df["menu_id"] = menu_encoder.fit_transform(df["CLICKPATH"])

# 时间特征
df["weekday"] = df["Date"].dt.weekday
df["month"] = df["Date"].dt.month
df["year"] = df["Date"].dt.year

# 3. 构建训练集
# 添加负样本（未点击的菜单）
# 这里简化处理，实际应根据业务需求调整采样比例
all_menus = df["menu_id"].unique()
user_dates = df[["user_id", "Date"]].drop_duplicates()

full_data = []
for _, row in user_dates.iterrows():
    user = row["user_id"]
    date = row["Date"]
    
    # 获取该用户当天实际点击的菜单
    clicked_menus = df[(df["user_id"] == user) & (df["Date"] == date)]["menu_id"]
    
    # 随机采样未点击的菜单作为负样本
    unclicked = np.setdiff1d(all_menus, clicked_menus)
    sampled_unclicked = np.random.choice(unclicked, size=min(20, len(unclicked)), replace=False)
    
    # 合并正负样本
    for menu in clicked_menus:
        full_data.append([user, date, menu, 1])  # 1表示点击过
    for menu in sampled_unclicked:
        full_data.append([user, date, menu, 0])  # 0表示未点击

train_df = pd.DataFrame(full_data, columns=["user_id", "Date", "menu_id", "clicked"])

# 合并原始特征
train_df = train_df.merge(df[["user_id", "Date", "menu_id", "weekday", "month", "year"]], 
                         on=["user_id", "Date", "menu_id"], how="left")

# 4. 特征分组
features = ["user_id", "menu_id", "weekday", "month", "year"]
target = "clicked"

# 5. 划分数据集
X_train, X_val, y_train, y_val = train_test_split(
    train_df[features], train_df[target], test_size=0.2, random_state=42
)

# 6. 训练LightGBM模型
params = {
    "objective": "binary",
    "metric": "auc",
    "num_leaves": 31,
    "learning_rate": 0.05,
    "feature_fraction": 0.9,
    "bagging_fraction": 0.8,
    "verbose": -1
}

model = lgb.LGBMClassifier(**params)
model.fit(X_train, y_train,
          eval_set=[(X_val, y_val)],
          callbacks=[lgb.early_stopping(stopping_rounds=10)])

# 7. 预测函数
def get_top10_menus(user_id, date):
    # 转换输入格式
    date = pd.to_datetime(date)
    weekday = date.weekday()
    month = date.month
    year = date.year
    
    # 生成所有菜单候选集
    all_menus = menu_encoder.classes_
    user_code = user_encoder.transform([user_id])[0]
    
    # 构建预测数据集
    pred_data = pd.DataFrame({
        "user_id": [user_code] * len(all_menus),
        "menu_id": menu_encoder.transform(all_menus),
        "weekday": [weekday] * len(all_menus),
        "month": [month] * len(all_menus),
        "year": [year] * len(all_menus)
    })
    
    # 预测点击概率
    probs = model.predict_proba(pred_data[features])[:, 1]
    
    # 获取Top10菜单
    top10_idx = np.argsort(probs)[-10:][::-1]
    return menu_encoder.inverse_transform(pred_data.iloc[top10_idx]["menu_id"])

# 使用示例
user_id = "u1"
date = "2025-03-06"
top_menus = get_top10_menus(user_id, date)
print(f"Top10推荐菜单：{top_menus}")