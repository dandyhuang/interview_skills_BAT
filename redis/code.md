# redis代码详解

### BitMap实现

```c++
class BitMap
{
public:
	BitMap(size_t Size = 1024)
	{
		_array.resize(Size/32+1);
	}

	~BitMap()
	{}
public:
	//将这个数存在的状态置为1
	void Set(const size_t& value)
	{
		size_t index = value>>5;//相当于除以32，取value在数组中的索引
		size_t bit = value % 32;//判断在32位中的哪一位
		_array[index] |= (1<<bit);
	}
	//将这个数不存在的状态置为0
	void Reset(const size_t& value)
	{
		size_t index = value>>5;
		size_t bit = value % 32;
		_array[index] &= (~(1<<bit));//~按位取反
	}
	//测试某个数是否出现过
	bool Test(const size_t& value)
	{
		size_t index = value>>5;
		size_t bit = value % 32;
		return (_array[index] & (1<<bit));
	}
private:
	vector<size_t> _array;
};
```



