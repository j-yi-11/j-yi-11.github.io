# neural network basis

```python
import numpy as np
import torch
from torch.utils.data import DataLoader, Dataset
import torch.nn as nn
import torch.optim as optim
from sklearn.model_selection import train_test_split

# 定义 Dataset 类
class CustomDataset(Dataset):
    def __init__(self, X, Y):
        self.X = torch.tensor(X, dtype=torch.float32)
        self.Y = torch.tensor(Y, dtype=torch.float32)
    
    def __len__(self):
        return self.X.shape[1]  # 样本数量
    
    def __getitem__(self, idx):
        return self.X[:, idx], self.Y[:, idx]

# 定义 Model 类
class NeuralNetworkModel(nn.Module):
    def __init__(self, n_x, n_h, n_y):
        super(NeuralNetworkModel, self).__init__()
        self.fc1 = nn.Linear(n_x, n_h)
        self.fc2 = nn.Linear(n_h, n_y)
    
    def forward(self, X):
        A1 = torch.tanh(self.fc1(X))
        A2 = torch.sigmoid(self.fc2(A1))
        return A2

# 定义训练函数
def train_model(model, dataloader, num_epochs=10000, learning_rate=0.01):
    optimizer = optim.Adam(model.parameters(), lr=learning_rate)
    loss_fn = nn.BCELoss()
    
    for epoch in range(num_epochs):
        epoch_loss = 0
        for X_batch, Y_batch in dataloader:
            optimizer.zero_grad()  # 清空梯度
            
            outputs = model(X_batch)  # 前向传播
            loss = loss_fn(outputs, Y_batch)  # 计算损失
            
            loss.backward()  # 反向传播
            optimizer.step()  # 更新权重
            
            epoch_loss += loss.item()
        
        if epoch % 1000 == 0:
            print(f'Epoch {epoch}, Loss: {epoch_loss}')

# 生成随机数据用于测试
np.random.seed(1)
n_features = 5  # 假设数据集有5个特征
X = np.random.randn(n_features, 400)  # 5个特征，400个样本
Y = (np.sum(X, axis=0) > 0).astype(int).reshape(1, 400)  # 生成标签

# 使用train_test_split函数划分训练集和测试集 (80% 训练，20% 测试)
X_train, X_test, Y_train, Y_test = train_test_split(X.T, Y.T, test_size=0.2, random_state=42)

# 转置数据，确保每个样本为一列
X_train = X_train.T
X_test = X_test.T
Y_train = Y_train.T
Y_test = Y_test.T

# 数据集和 DataLoader
train_dataset = CustomDataset(X_train, Y_train)
test_dataset = CustomDataset(X_test, Y_test)

train_loader = DataLoader(train_dataset, batch_size=64, shuffle=True)
test_loader = DataLoader(test_dataset, batch_size=64, shuffle=False)

# 初始化模型
n_x = n_features  # 输入特征数量
n_h = 4           # 隐藏层神经元数
n_y = 1           # 输出一个结果（0或1）

model = NeuralNetworkModel(n_x=n_x, n_h=n_h, n_y=n_y)

# 训练模型
train_model(model, train_loader, num_epochs=100)

# 测试模型预测
def evaluate_model(model, dataloader):
    model.eval()  # 设置模型为评估模式
    correct = 0
    total = 0
    with torch.no_grad():
        for X_batch, Y_batch in dataloader:
            outputs = model(X_batch)
            predicted = outputs.round()  # 预测的输出
            total += Y_batch.size(1)  # 总样本数
            correct += (predicted == Y_batch).float().sum().item()  # 计算正确的预测数
    
    accuracy = 100 * correct / total
    return accuracy

# 评估模型在测试集上的表现
test_accuracy = evaluate_model(model, test_loader)
print(f'Test Accuracy: {test_accuracy}%')
```

