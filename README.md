# Contrastive Loss

NLP Task에 Contrastive Loss를 적용하기 위하 찾아본 코드들을 정리한 repo
   
   
## Contrastive Loss 특징

일반적인 CrossEntropy Loss는 hidden_size -> num_class로 Linear 레이어를 통과시켜 각 Class별 Logits 값을 구한 후 이 Logits과 Label을 이용하여 Loss를 구함. Logits, Label -> Loss

ex)   
입력   
Logits : [batch_size, num_class]   
Label : [batch_size]      
↓    
출력   
Loss   
 
Contrastive Loss는 직접적으로 Label을 맞추는 것보다는 각 Label 별로 샘플들의 hidden_size가 잘 나눠지도록 함.    
예를 들어 Label A,B,C인 샘플이 각각 10개 있다면, Label이 A인 각 샘플들의 hidden_size는 더 유사하게, Label B, C인 샘플들과는 더 다르게 유도한다.

이러한 유사성, 다름의 정의를 무엇으로 하느냐에 따라 유클리드 거리, 마할라노비스 거리, 코사인 유사도 등 다양한 방법을 사용할 수 있다.

ex)   
입력   
Embedding : [batch_size, hidden_size]   
Label : [batch_size]   
↓   
출력   
Loss

유사성, 거리 기준 외에도 입력 데이터를 어떻게 구성하는지에 대해서 여러가지 전략이 있는 것으로 보인다.   
기본적으로 Batch 단위로 Loss를 구하는 것은 일반적인 CE Loss와 같다.   
하지만 CE Loss는 결국 Sample - Class의 관계가 보다 직접적이기 때문에 batch size의 영향이 상대적으로 적지만
Contrastive Loss의 경우에는 Mini batch 내의 Class별 Embedding의 분포를 보기 때문에 batch size나 클래스 내 분포의 영향이 더 큰 것으로 보인다.

때문에 입력 데이터를 어떻게 구성할 것인가도 중요한 문제가 된다.

아래는 일반적인 분류라기보다는 여러가지 테스트를 하면서 내가 직관적으로 이해한 기준으로 나눈 분류이다.

### single_input
비교적 간단한 구조이다.
일반적인 CCE Loss와 마찬가지로 그냥 모델을 통해 나온 값을 배치 단위로 넣어주면 된다.

### multi_input
여러 전략이 있을 수 있겠지만 모델의 dropout를 노이즈로 적용시켜서 같은 데이터에 다른 dropout probability를 적용시켜서 같은 샘플에 대해 노이즈가 적용된 데이터를 만들고, 그걸 Loss를 구하는데 활용하는 방식이 일반적인 것 같다.

모델의 dropout p를 list 형태로 준 후 for문을 통해 같은 샘플에 다른 dropout p가 적용된 데이터들을 stack한 후에 Loss를 구하는 방식이다.
때문에 단순히 모델의 특정 결과값만 뽑아서 Loss만 바꾸면 되는 방식과 달리 Trainer의 구조를 바꿔줘야한다.   
데이터는 아래와 같이 구성 한 후 label과 함께 입력값으로 넣어 Loss를 구한다.
[batch size, num dropout p, hiddensize]
   
어떤 전략이 더 효과적인지는 Task나 데이터에 따라 다른 것 같다.   
ABSA Task를 Encoder-Decoder 모델로 각 Aspect와 Polarity가 조합된 것을 Special Token으로 추가하여 Test를 해봤을 때는 single input 방식에 코사인 유사도를 적용했을 때 가장 좋은 결과를 얻을 수 있었다.
