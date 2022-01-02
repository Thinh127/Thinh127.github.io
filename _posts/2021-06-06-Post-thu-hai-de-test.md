---
layout: post

title: Lịch làm việc trong một ngày

date: 2021-01-01 00:15:00 +/-0127

categories: ["getting start", "test post"]

tags: ["should not be seen"]

comments: true

math: true

toc: true

published: true
---
# 1. Tiêu đề một

Đây là lịch làm việc đầu tiên trong bài viết. Phần kế tiếp sẽ hiển thị trong tiêu đề với đề mục số 2.

# 2. Tiêu đề hai

Hello tiêu đề hai. Sau đây ta sẽ tiếp tục đến với tiêu đề số 3 nhé. Nhanh thôi. Vì tôi sẽ không làm bạn mất thêm thời gian nên đề tiêu đề số 3 sẽ bắt đầu ngay sau đây. Vâng, không chờ đợi thêm 1 giây nào nữa thì tiêu đề 3 sẽ được bắt đầu như chúng ta dự định. Và sau đây là sự bắt đầu của tiêu đề 3, các bạn sẽ không còn phải chờ đợi thêm một giây nào nữa rồi. À, cảm ơn vì bạn đã cố gắng theo dõi đến phút giây này. Không phụ sự kiên nhẫn đó, tiêu đề 3, phần ngay tiếp sau đây là xuất hiện trong sự vui mừng của đọc giả. Đặc biệt, sau khi tiêu đề 3 bắt đầu tôi sẽ nói về một chủ đề rất hay mà tôi sẽ bàn sau. Ok. Và đây là tiêu đề 3.

# 3. Tiêu đề 3
Trong tiêu đề 3 chúng ta sẽ có 3 phần tiêu đề con như sau:

## a. Tiêu đề con a

Bạn thấy tiêu đề nhỏ này trong toc (table of content) chứ?

## b. Tiêu đề con b

Tiêu đề con cuối cùng, bạn có thấy nó trong toc?

Đây là một bức ảnh  chốt bài của tiêu đề 3
![Anh tu unsplash](https://unsplash.com/blog/content/images/max/1200/1-ah4VbDl5zLC87Tw2L52x-A.jpeg)

# 4. Phần này thử công thức toán nhé
Công thức tổng độ lỗi bình phương:
$J(w)=\dfrac{1}{2}\sum_{i=1}^N(y^{(i)}-z^{(i)})^2$
## Mục nhỏ dành cho code

```python
import numpy as np

class Perceptron(object):
	"""Perceptron classifier
	Parameters
	-----------
	eta: float
			Learning rate (0 < eta < 1)
	n_iter: int
			Passes over the training dataset
	random_state: int
			Random number generator seed for reproducible

	Attributes
	-----------
	w_: 1d-array
		Weights after fitting
	errors_: list
		Number of misclassification in each epoch
	"""
	def __init__(self, eta=0.01, n_iter=50, random_state=1):
		self.eta = eta
		self.n_iter = n_iter
		self.random_state = random_state

	def fit(self, X, y):
		"""Fit training data.
		Parameters
		-----------
		X: array-like, shape = [n_examples, n_features]
			Training vectors, where n_examples is the number of examples
			n_features is the number of features
		y: array-like, shape = [n_examples]
			Target values
		
		Returns
		--------
		self: object
		"""
		rgen = np.random.RandomState(self.random_state)
		self.w_ = rgen.normal(loc=0, scale=0.01, size=1+ X.shape[1])
		self.errors_ = []
		
		for _ in range(self.n_iter):
			errors = 0
			for xi, target in zip(X, y):
				update = self.eta * (target - self.predict(xi))
				self.w_[1:] += update * xi
				self.w_[0] += update
				errors += int(update != 0)
			self.errors_.append(errorso)
		return self

	def net_input(self, X):
		"""Calculate net input"""
		return np.dot(X, self.w_[1:]) + self.w_[0]
	
	def predict(self, X):
		"""Return class label after unit step"""
		return np.where(self.net_input(X) >= 0, 1, -1)
```
	
