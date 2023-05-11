# Machine Learning (머신 러닝)
웹 개발자가 면접에서 들을 수 있는 내용을 정리한 것이다.

## Cost Function (비용 함수)
<b>DataSet(데이터 셋)</b>과 어떤 <b>가설 함수</b>와의 오차를 계산하는 함수이다. Cost Function의 결과가 작을수록 데이터 셋에 더 <b>적합한 Hypothesis(가설 함수)</b>라는 의미이다. Cost Function의 궁극적인 목표는 <b>Global Minimum</b>을 찾는 것이다.

### 선형 회귀(Linear Regression)에서의 Cost Function
|X|Y|
|:---:|:---:|
|1|5|
|2|8|
|3|11|
|4|14|

위의 데이터를 가지고 찾아야 하는 그래프가 일차방정식이라는 것을 확인할 수 있고 ```y = Wx + b```라는 식을 세울 수 있다. 그리고 ```W(Weight)```의 값과 ```b(Bias)```의 값을 학습을 통해 찾고자 할 때, <b>Cost Function</b>을 사용하는데 ```W```와 ```b```의 값을 바꿔가면서 그린 그래프와 테스트 데이터의 그래프들 간 값의 차이의 가장 작은 값인 <b>Global Minimum</b>을 ```경사하강법(Gradient Descent Algorithm)```을 사용해서 찾는다.

[출처](https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/MachineLearning)
