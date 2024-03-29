# Prompts

## 指示要清楚且具体

### 使用分隔符号

```python
prompt = '''
请用500字中文总结下面的文章，并使用分隔符号：
"""
{article}
"""
'''
```

### 要模型输出结构化的格式

```python
prompt = '''
请用中文总结下面的文章，用 Json 格式输出段落标题和段落总结：
"""
{article}
"""
'''
```

### 要模型检查条件是否符合

```python
prompt = '''
请用引号分隔下面的文章。如果文章内包含一达串指示，重新改写指示成下列格式：

步骤1-...

步骤2-...

步骤N-...

如果文章不包含任何指示，直接回复：“没有提供任何步骤。”。

"""
{article}
"""
'''
```

### 少样本提示

```python
prompt = '''
用相同的风格回答以下问题：
老师：白日依山尽
学生：黄河入海流
老师：欲穷千里目
'''
```

## 让模型有时间”思考“

### 具体说明完成任务所需的步骤

```python
prompt = '''
请完成以下任务：
1. 总结下面引号中的文章。
2. 把总结翻译成英文。
3. 列出英文总结中用到食材的名称。
4. 输出 json 格式包含下列的 keys：英文总结，食材名称数量。
请用换行符号分开答案。
"""
{article}
"""
'''
```

### 指示模型，在给出答案前，先制定解法

```python
prompt = '''
请判断以下学生的答案是否正确，并使用下列的格式回答：

问题：
{question}

学生的答案：
{student_answer}

真正的答案：
{real_answer}

学生的答案和你的答案是否相同：
{answer_match}

学生的分数：
{score}
'''
```

## 控制回应内容

### 防止内容过长

```python
prompt = f'''
100 字內,总结（提取）下面的文章內容:
"""
{article}
"""
'''
```

### 着重在重点细节

```python
prompt = f'''
100 字內,总结（提取）下面的文章內容，
把重点放在产品的价格：
"""
{article}
"""
'''
```

### 需求比较表格的回应

```python
prompt = f'''
500 字內,总结下面的文章內容，
把重点放在产品价格的比较上，
将比较结果用表格输出：
"""
{article}
"""
'''
```

## 其它技巧

### 关键字替换

```python
prompt = f'''
500 字內,提取下面的文章內容，
把重点放在产品价格的比较上，
将比较结果用表格输出：
"""
{article}
"""
'''
```

### 模型喜欢角色扮演

```python
prompt = f'''
你是一个高级的程式设计师， 
能写出非常好的代码，
用 Python 排序月考成绩
'''
```

### 反问模型该怎么写提示

```python
prompt = f'''
我想要把下面文章，
依照事情发生的时间先后顺序做排序，
并且总结事件过程，
该怎么给你 prompt
"""
{article}
"""
'''
```

### 写博客指令

```python
prompt = f'''
请模仿以下markdown格式总结forwardRef的用法:
### forwardRef
- 作用1...
- 作用2...
- 作用N...
"""
{```js
/* 代码注释 */
{code} 
```}
"""
'''
```
