

本周学习了词向量的相关内容，主要学习了跳字和连词理论推导，并理解和运行跳字模型的代码实现。



## 词向量

### 离散表示 one-hot

将一段文本的不重复的词汇进行one-hot编码。
但是使用one-hot并非一个很好的选择 主要原因是：1、如果词汇数量很多，那么one-hot的维度就会很大。2、数据稀疏。3、无法准确的表示不同词之间的相似度。

### Skip-Gram（跳字）模型

跳字模型假设基于某个词开生成它在文本序列周围的词。

以"The man loves his son"为例。以"love"为中心词，背景窗口大小为2，跳字模型关心的是给中心词"loves" 生成与它距离不超过2个词的背景词"The" "man" "his" "son"的条件概率。
$$
P(the,man,his,son|loves)
$$
假设给定的中心词的情况下，背景词的生成是相互独立的，那么上式可以改写为
$$
P(the|loves)P(man|loves)P(his|loves)P(son|loves)
$$
在跳字模型中，每个词被表示为两个d维向量，（d 维 为不重复词汇的数量）用来计算条件概率。

假设这个词在词典中的索引为i，当它为中心词向量表示为$v_i\in\mathbb{A}^d$背景词表示为$u_i\in \mathbb{A}^d$.设中心词$w_c$在词典中的索引为c。背景词的索引为$o$ 给定中心词生成背景词的条件概率可以通过对向量内积做softmax运算得到。
$$
P(w_o|w_c)=\frac{exp(u_o^Tv_c)}{\sum_{i\in v}exp(u_i^Tv_c)}
$$
其中词典索引集$V=\{0,1,2...,|v|-1\}$ 假设给定一个长度为T的文本序列，设时间步t的词为$w^t$ 假设给定中性词的情况下背景词的生成相互独立，当背景窗口大小为m时，跳字模型的似然函数即给定中心词所生成的所有背景词的概率
$$
\prod_{t=1}^{T}\prod_{-m\le{j}\le{m}}P(w^{t+j}|w^{(t)})
$$

需要求上式的最大值，等价于求下式的最小值
$$
-\sum_{t=1}^{T}\sum_{-m\le{j}\le{m},{j}\neq0}logP(w^{t+j}|w^t)
$$
也就是求上式梯度即可

根据定义可以求出
$$
logP(w_o|w_c)=u_o^Tv_c-log({\sum_{i\in{\mathcal{V}}}exp(u_i^Tv_c)})
$$
求导得到结果为
$$
\frac{\partial{logP(w^{t+j}|w^t})}{\part{v_c}}=u_o-{\sum_{j\in{\mathcal{V}}}P(w_j|w_c)}u_j
$$
从推导结果可以看出他的计算需要词典中所有词以$w_c$为中心词的条件概率。

### 连续词袋模型

连续词袋模型与跳字模型类似，概念相反，跳字模型是使用中心词预测周围词，连续词袋模型利用周围词预测中性词。

也就是
$$
P(love|the,man,his,son)
$$


因为连续词袋模型的背景词有多个，将这些背景词向量取平均，然后使用和跳字模型一样的方法来计算条件概率。设$v_i\in{\mathbb{R}^d}$和$u_i\in {\mathbb{R}^d}$分别表示词典中索引为i的词作为背景词和中心词的向量。设中心词$w_c$在词典中的索引为c，背景词$w_{o_1}...w_{o_{2m}}$ 在词典中的索引为$o_1....o_{2m}$，那么给定背景词生成中心词的条件概率为
$$
P(w_c|w_{o_1}...w_{o_{2m}})=\frac{exp(\frac{1}{2m}u_c^T(v_{o_1}+...+v_{o_2m}))}{\sum_{i\in{v}}exp(\frac{1}{2m}u_i^T(v_{o_i}+...+v_{o_{2m}}))}
$$
给出一个长度为T的文本序列，设时间步t的词为$w^t$,背景窗口大小为m。连续词袋模型的似然函数是由背景词生成任一中心词的概率为
$$
\prod_{t=1}^TP(w^t|w^{t-m}...w^{t}...w^{t+m})
$$
​	求偏导得到
$$
\frac{\part{logP(w_c|\mathcal{W_0})}}{\part{v_{o_i}}}=\frac{1}{2m}(u_c-\sum_{j\in{\mathcal{V}}}P(w_j|{\mathcal{W_o}})u_j)
\\
\\
\\
\mathcal{W}=\{w_{o_1},....w_{o_{2m}}\}
$$
从上式也可以看出需要对词典中的所有词进行计算。

## 进行近似训练

以跳字模型为例，使用softmax运算得到给定中性词$w_c$生成背景词$w_o$的条件概率
$$
P(w_o|w_c)=\frac{exp(u_o^Tv_c)}{\sum_{i\in v}exp(u_i^Tv_c)}
$$
损失函数为
$$
-logP(w_o|w_c)=-u_o^Tv_c+log({\sum_{i\in{\mathcal{V}}}exp(u_i^Tv_c)})
$$
由于softmax考虑到了背景词可能是词典V中任意一个词，上面的损失函数包含了词典大小数目项的累加。计算代价过大，因此采用近似训练法“负采样”或者“层序softmax”进行训练。



### 负采样

负采样修改了原来的目标函数。给定中心词$w_c$的一个背景窗口，我们把背景词$w_o$出现在该背景窗口看作一个事件，并将该事件的概率计算为
$$
P(D=1|w_c,w_o)=\sigma(u_o^Tv_c)
$$
其中$\sigma$函数与sigmoid激活函数的定义相同


$$
\sigma(x)=\frac{1}{1+exp(-x)}
$$

```python
    def forward_noise(self, size, N_SAMPLES):
        noise_dist = self.noise_dist
        #从词汇分布中采样负样本
        noise_words = torch.multinomial(noise_dist,
                                        size * N_SAMPLES,
                                        replacement=True)
        noise_vectors = self.out_embed(noise_words).view(
            size, N_SAMPLES, self.n_embed)
        return noise_vectors
```

我们先考虑最大化文本序列中所有该事件的联合概率来训练词向量。具体来说，给定一个长度为T的文本序列，设时间步t的词为$w^t$且背景窗口大小为$m$考虑最大化联合概率
$$
\prod_{t=1}^T\prod_{-m\le{j}\le{m}}P(D=1|(w^t,w^{t+j}))
$$
然而，以上模型中包含的事件仅考虑了正样本，则就导致了当所有词向量相等且为无穷大时以上的联合概率才能最大化为1。负采样通过采样添加负样本使目标函数变得更加有意义。设背景词$w_o$出现在中心词$w_c$的一个背景窗口为事件P，我们根据分布P(w)，采样K个未出现在该背景窗口中的词，也就是噪声词。设噪声词$w_k(k=1,...,K)$不出现在中心词$w_c$的背景为事件$N_k$.假设同时含有正样本和负样本的事件为P,$N_1,...N_k$相互独立，负采样将以上需要最大化的仅考虑正样本的联合概率改写为
$$
\prod_{t=1}^T\prod_{-m\le{j}\le{m}}P(w^{t+j}|w^t)
$$
其中
$$
P(w^{t+j}|w^t) = P(D=1|w^t,w^{t+j})\prod_{k=1,w_k\sim{p_{(w)}}}P(D=0|w^t,w_k)
$$
对于损失函数：
$$
-logP(w^{(t+j)}|w^t)=-log{\sigma(u_{i_{t+j}}v_{i_t})}-\sum_{k=1,w_k\sim{P(w)}}^Klog\sigma(-u_{h_k}^Tv_{i_t})
$$

```python
#定义损失函数
class NegativeSamplingLoss(nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self, input_vectors, output_vectors, noise_vectors):
        BATCH_SIZE, embed_size = input_vectors.shape
        #将输入词向量与目标词向量作维度转化处理
        input_vectors = input_vectors.view(BATCH_SIZE, embed_size, 1)
        output_vectors = output_vectors.view(BATCH_SIZE, 1, embed_size)
        #目标词损失
        test = torch.bmm(output_vectors, input_vectors)
        out_loss = torch.bmm(output_vectors, input_vectors).sigmoid().log()
        out_loss = out_loss.squeeze()
        #负样本损失
        noise_loss = torch.bmm(noise_vectors.neg(),
                               input_vectors).sigmoid().log()
        noise_loss = noise_loss.squeeze().sum(1)
        #综合计算两类损失
        return -(out_loss + noise_loss).mean()
```

这时的损失函数已经与词典的大小没有了关系，只与K具有线性关系。



使用Pytorch实现skip-gram

数据处理

```python
def process(text: str, appearance_rate: int):
    """
    处理文本 将文本转换成为单词列表 并去除低于appearance_rate的单词
    """
    # 将所有的词汇转换成小写
    text = text.lower()
    # 将句子转换成词汇
    words = text.split()
    # 统计词汇出现的次数
    word_counts = Counter(words)
    # 去除低于appearance_rate的词汇
    trimmed_words = [
        word for word in words if word_counts[word] > appearance_rate
    ]
    return trimmed_words
```



模型定义

```python
def get_train_words(word_freqs, t=0.1):
    assert t > 0
    train_words = [
        word for word, rate in zip(word_freqs.keys(), word_freqs.values())
        if random.uniform(0, 1) > (1 - math.sqrt(t / rate))
    ]
    return train_words


print(get_train_words(word_freqs, t=0.2))

train_words = get_train_words(word_freqs, t=0.2)

word_freqs = np.array(list(word_freqs.values()))
print(word_freqs)
unigram_dist = word_freqs / word_freqs.sum()
noise_dist = torch.from_numpy(unigram_dist**(0.75) /
                              np.sum(unigram_dist**(0.75)))  # 论文中为3/4

print(noise_dist)
```

定义网络模型

```python
class SKipGramNet(nn.Module):
    def __init__(self, n_vocab, n_embed, noise_dist):
        super().__init__()
        self.n_vocab = n_vocab
        self.n_embed = n_embed
        self.noise_dist = noise_dist
        #定义词向量层
        self.in_embed = nn.Embedding(n_vocab, n_embed)
        self.out_embed = nn.Embedding(n_vocab, n_embed)
        #词向量层参数初始化
        self.in_embed.weight.data.uniform_(-1, 1)
        self.out_embed.weight.data.uniform_(-1, 1)

    #输入词的前向过程
    def forward_input(self, input_words):
        input_vectors = self.in_embed(input_words)
        return input_vectors

    #目标词的前向过程
    def forward_output(self, output_words):
        output_vectors = self.out_embed(output_words)
        return output_vectors
        #负样本词的前向过程
    def forward_noise(self, size, N_SAMPLES):
        noise_dist = self.noise_dist
        #从词汇分布中采样负样本
        noise_words = torch.multinomial(noise_dist,
                                        size * N_SAMPLES,
                                        replacement=True)
        noise_vectors = self.out_embed(noise_words).view(
            size, N_SAMPLES, self.n_embed)
        return noise_vectors
```

训练可视化结果![](D:\MarkDown\DeepLearning\img\demo.png)

可以看出 "book","music","movie" 距离很近 " animal","cat","like","hate","dog" 距离很近 从训练的文本中也可以看出词汇之间的关系训练的相对还是不错的。



