# 通用变量和参数

source: `{{ page.path }}`

## WTSVariant

WTSVariant是一个通用数据容器, 设计目标是Json的Value类, 和Json不同的地方在于, WTSVariant满足WT系统内的派生关系, 可以通过引用计数管理数据, 从而减少数据复制

主要有 Object 和 Array 两种数据类型, Object类型用 WTSHashMap 存放, Array 类型用 WTSArray 存放

```cpp
class WTSVariant : public WTSObject
{
public:
    // 数值容器, 存放子节点数据
	typedef WTSArray					ChildrenArray;
    // map容器, 键: string, 值: WTSObject*
	typedef WTSHashMap<std::string>		ChildrenMap;
    // 子节点的全部键名
	typedef std::vector<std::string>	MemberNames;

	typedef enum
	{
		VT_Null,
		VT_Array,
		VT_Int32,
		VT_Uint32,
		VT_Int64,
		VT_Uint64,
		VT_String,
		VT_Real,
		VT_Boolean,
		VT_Object
	} ValueType;

private:
	union ValueHolder
	{
		std::string*	_string;

		ChildrenMap*	_map;
		ChildrenArray*	_array;
	};
	ValueHolder	_value;
	ValueType	_type;

protected:
	WTSVariant() :_type(VT_Null){}

private:
    // 函数重载, 创建不同类型 WTSVariant 对象
	static WTSVariant* create(int32_t i32)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Int32;
		char s[32] = { 0 };
		sprintf(s, "%d", i32);
		ret->_value._string = new std::string(s);

		return ret;
	}

	static WTSVariant* create(uint32_t u32)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Uint32;
		char s[32] = { 0 };
		sprintf(s, "%u", u32);
		ret->_value._string = new std::string(s);
		return ret;
	}

	static WTSVariant* create(int64_t i64)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Int64;
		char s[32] = { 0 };
		sprintf(s, INT64_FMT, i64);
		ret->_value._string = new std::string(s);
		return ret;
	}

	static WTSVariant* create(uint64_t u64)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Uint64;
		char s[32] = { 0 };
		sprintf(s, UINT64_FMT, u64);
		ret->_value._string = new std::string(s);
		return ret;
	}

	static WTSVariant* create(double _real)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Real;
		char s[32] = { 0 };
		sprintf(s, "%f", _real);
		ret->_value._string = new std::string(s);
		return ret;
	}

	static WTSVariant* create(const char* _string)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_String;
		ret->_value._string = new std::string(_string);
		return ret;
	}

	static WTSVariant* create(bool _bool)
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Boolean;
		ret->_value._string = new std::string(_bool ? "true" : "false");
		return ret;
	}

public:
	// Object和Array类型单独创建
	static WTSVariant* createObject()
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Object;
		ret->_value._map = ChildrenMap::create();
		return ret;
	}

	static WTSVariant* createArray()
	{
		WTSVariant* ret = new WTSVariant();
		ret->_type = VT_Array;
		ret->_value._array = ChildrenArray::create();
		return ret;
	}

	// 判断Object类型对象是否存在 key
	bool		has(const char* key) const
	{
		if (_type != VT_Object)
			return false;

		auto it = _value._map->find(key);
		if (it == _value._map->end())
			return false;

		return true;
	}

	// 将数据转换为指定数值类型
	int32_t		asInt32() const
	{
		switch (_type)
		{
		case VT_Null:
			return 0;
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
			return _value._string ? (int32_t)atof(_value._string->c_str()) : 0;
		default:
			return 0;
		}
	}

	uint32_t	asUInt32() const
	{
		switch (_type)
		{
		case VT_Null:
			return 0;
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
			return _value._string ? (uint32_t)atof(_value._string->c_str()) : 0;
		default:
			return 0;
		}
	}

	int64_t		asInt64() const
	{
		switch (_type)
		{
		case VT_Null:
			return 0;
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
			return _value._string ? strtoll(_value._string->c_str(), NULL, 10) : 0;
		default:
			return 0;
		}
	}

	uint64_t	asUInt64() const
	{
		switch (_type)
		{
		case VT_Null:
			return 0;
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
			return _value._string ? strtoull(_value._string->c_str(), NULL, 10) : 0;
		default:
			return 0;
		}
	}

	double		asDouble() const
	{
		switch (_type)
		{
		case VT_Null:
			return 0.0;
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
			return _value._string ? strtod(_value._string->c_str(), NULL) : 0.0;
		default:
			return 0.0;
		}
	}

	std::string	asString() const
	{
		switch (_type)
		{
		case VT_Null:
			return "";
		case VT_Int32:
		case VT_Uint32:
		case VT_Int64:
		case VT_Uint64:
		case VT_Real:
		case VT_String:
		case VT_Boolean:
			return _value._string ? *_value._string : "";
		default:
			return "";
		}
	}

	const char* asCString() const
	{
		if (_type != VT_Object && _type != VT_Array && _value._string != NULL)
			return _value._string->c_str();

		return "";
	}

	bool asBoolean() const
	{
		if (_value._string)
		{
			return _value._string->compare("true") == 0;
		}

		return false;
	}

	// 根据名称获取指定数据并强制类型转换
	int32_t		getInt32(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asInt32();

		return 0;
	}

	uint32_t	getUInt32(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asUInt32();

		return 0;
	}

	int64_t		getInt64(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asInt64();

		return 0;
	}

	uint64_t	getUInt64(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asUInt64();

		return 0;
	}

	double		getDouble(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asDouble();

		return 0.0;
	}

	std::string	getString(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asString();

		return "";
	}

	const char* getCString(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asCString();

		return "";
	}

	bool getBoolean(const char* name) const
	{
		WTSVariant* p = get(name);
		if (p)
			return p->asBoolean();

		return false;
	}

	// 根据名称获取数据指针
	WTSVariant* get(const char* name) const
	{
		if (_type != VT_Object)
			return NULL;

		if (_value._map == NULL)
			return NULL;

		WTSVariant* ret = static_cast<WTSVariant*>(_value._map->get(name));
		return ret;
	}

	WTSVariant* get(const std::string& name) const
	{
		if (_type != VT_Object)
			return NULL;

		if (_value._map == NULL)
			return NULL;

		WTSVariant* ret = static_cast<WTSVariant*>(_value._map->get(name));
		return ret;
	}

	// 根据索引获取数据指针
	WTSVariant* get(uint32_t idx) const
	{
		if (_type != VT_Array)
			return NULL;

		if (_value._array == NULL)
			return NULL;

		WTSVariant* ret = static_cast<WTSVariant*>(_value._array->at(idx));
		return ret;
	}

	// 在Object类型对象的map中添加数据
	bool append(const char* _name, const char* _string)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_string);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, int32_t _i32)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_i32);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, uint32_t _u32)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_u32);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, int64_t _i64)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_i64);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, uint64_t _u64)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_u64);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, double _real)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_real);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, bool _bool)
	{
		if (_type != VT_Object)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSVariant* item = WTSVariant::create(_bool);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, WTSVariant *item, bool bAutoRetain = true)
	{
		if (_type != VT_Object || NULL == item)
			return false;
		if (_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		_value._map->add(_name, item, bAutoRetain);
		return true;
	}

	// 在Array类型对象的array中添加数据
	bool append(const char* _str)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_str);
		_value._array->append(item, false);
		return true;
	}

	bool append(int32_t _i32)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_i32);
		_value._array->append(item, false);
		return true;
	}

	bool append(uint32_t _u32)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_u32);
		_value._array->append(item, false);
		return true;
	}

	bool append(int64_t _i64)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_i64);
		_value._array->append(item, false);
		return true;
	}

	bool append(uint64_t _u64)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_u64);
		_value._array->append(item, false);
		return true;
	}

	bool append(double _real)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_real);
		_value._array->append(item, false);
		return true;
	}

	bool append(bool _bool)
	{
		if (_type != VT_Array)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSVariant* item = WTSVariant::create(_bool);
		_value._array->append(item, false);
		return true;
	}

	bool append(WTSVariant *item, bool bAutoRetain = true)
	{
		if (_type != VT_Array || NULL == item)
			return false;
		if (_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		_value._array->append(item, bAutoRetain);
		return true;
	}

	// 获取数据大小
	uint32_t size() const
	{
		if (_type != VT_Array && _type != VT_Object)
			return 0;

		else if (_type == VT_Array)
		{
			return (_value._array == NULL) ? 0 : _value._array->size();
		}
		else
		{
			return (_value._map == NULL) ? 0 : _value._map->size();
		}
	}

	// 获取map类型数据的所有键名
	MemberNames memberNames() const
	{
		MemberNames names;
		if (_type == VT_Object && _value._map != NULL)
		{
			auto it = _value._map->begin();
			for (; it != _value._map->end(); it++)
			{
				names.emplace_back(it->first);
			}
		}
		return names;
	}

	// 释放内存
	virtual void release()
	{
		if (isSingleRefs())
		{
			switch (_type)
			{
			case VT_Array:
				if (NULL != _value._array)
				{
					_value._array->release();
				}
				break;
			case VT_Object:
				if (NULL != _value._map)
				{
					_value._map->release();
				}
				break;
			default:
				if (NULL != _value._string)
				{
					delete _value._string;
				}
				break;
			}
		}
		WTSObject::release();
	}

	// 获取数据类型
	inline ValueType type() const{ return _type; }

	// 将 WTSVariant 转化为 WTSParams
	WTSParams* toParams()
	{
		WTSParams* ret = NULL;
		if (_type == VT_Array)
			ret = WTSParams::createArray();
		else if (_type == VT_Object)
			ret = WTSParams::createObject();
		else
			return NULL;

		if (_type == VT_Object && _value._map != NULL)
		{
			auto it = _value._map->begin();
			for (; it != _value._map->end(); it++)
			{
				WTSVariant* val = (WTSVariant*)it->second;
				const char* key = it->first.c_str();
				if (val->type() == VT_Object || val->type() == VT_Array)
				{
					ret->append(key, val->toParams(), false);
				}
				else
				{
					ret->append(key, val->asCString(), "");
				}
			}
		}
		else if (_type == VT_Array && _value._array != NULL)
		{
			ChildrenArray::Iterator it = _value._array->begin();
			for (; it != _value._array->end(); it++)
			{
				WTSVariant* val = (WTSVariant*)(*it);
				if (val->type() == VT_Object || val->type() == VT_Array)
				{
					ret->append(val->toParams(), false);
				}
				else
				{
					ret->append(val->asCString(), "");
				}
			}
		}
		else
		{
			ret->append(this->asCString(), "");
		}

		return ret;
	}

	// 判断类型
	bool isArray() const{ return _type == VT_Array; }
	bool isObject() const{ return _type == VT_Object; }
};
```

## WTSParams

和 `WTSVariant` 用法类似

```cpp
class WTSParams : public WTSObject
{
public:
	typedef WTSArray					ChildrenArray;
	typedef WTSHashMap<std::string>		ChildrenMap;
	typedef std::vector<std::string>	MemberNames;

	typedef enum
	{
		VT_Value,
		VT_Array,
		VT_Object
	}ValueType;

private:	
	union ValueHolder
	{
		std::string*	_string;
		ChildrenMap*	_map;
		ChildrenArray*	_array;
	};
	std::string	_desc;
	ValueHolder	_value;
	ValueType	_type;

protected:
	WTSParams():_type(VT_Value){}

private:
	// 函数重载, 创建对象
	static WTSParams* create(int32_t i32, const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		char s[32] = { 0 };
		sprintf(s, "%d", i32);
		ret->_value._string = new std::string(s);
		ret->_desc = desc;
		return ret;
	}

	static WTSParams* create(uint32_t u32, const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		char s[32] = { 0 };
		sprintf(s, "%u", u32);
		ret->_value._string = new std::string(s);
		ret->_desc = desc;
		return ret;
	}

	static WTSParams* create(double _real, const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		char s[32] = { 0 };
		sprintf(s, "%f", _real);
		ret->_value._string = new std::string(s);
		ret->_desc = desc;
		return ret;
	}

	static WTSParams* create(const char* _string, const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		ret->_value._string = new std::string(_string);
		ret->_desc = desc;
		return ret;
	}

	static WTSParams* create(bool _bool, const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		ret->_value._string = new std::string(_bool?"true":"false");
		ret->_desc = desc;
		return ret;
	}

public:
	// Object和Array对象单独创建
	static WTSParams* createObject(const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		ret->_type = VT_Object;
		ret->_value._map = ChildrenMap::create();
		ret->_desc = desc;
		return ret;
	}

	static WTSParams* createArray(const char* desc="")
	{
		WTSParams* ret = new WTSParams();
		ret->_type = VT_Array;
		ret->_value._array = ChildrenArray::create();
		ret->_desc = desc;
		return ret;
	}
	
	// 将数据转为指定类型
	int32_t		asInt32() const
	{
		switch(_type)
		{
		case VT_Value:
			return _value._string?atoi(_value._string->c_str()):0;
		default:
			return 0;
		}
	}

	uint32_t	asUInt32() const
	{
		switch(_type)
		{
		case VT_Value:
			return _value._string?strtoul(_value._string->c_str(), NULL, 10):0;
		default:
			return 0;
		}
	}

	double		asDouble() const
	{
		switch(_type)
		{
		case VT_Value:
			return _value._string?strtod(_value._string->c_str(), NULL):0.0;
		default:
			return 0.0;
		}
	}

	std::string	asString() const
	{
		switch(_type)
		{
		case VT_Value:
			return _value._string?*_value._string:"";
		default:
			return "";
		}
	}

	const char* asCString() const
	{
		if(_type == VT_Value && _value._string != NULL)
			return _value._string->c_str();

		return "";
	}

	bool asBoolean() const
	{
		if(_type == VT_Value && _value._string)
		{
			return _value._string->compare("true") == 0;
		}

		return false;
	}

	// 根据名称获取数据并做类型转换
	int32_t		getInt32(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asInt32();

		return 0;		
	}

	uint32_t	getUInt32(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asUInt32();

		return 0;	
	}

	double		getDouble(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asDouble();

		return 0.0;	
	}

	std::string	getString(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asString();

		return "";	
	}

	const char* getCString(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asCString();

		return "";	
	}

	bool getBoolean(const char* name) const
	{
		WTSParams* p = get(name);
		if(p)
			return p->asBoolean();

		return false;	
	}

	// 根据名称获取对象指针
	WTSParams* get(const char* name) const
	{
		if(_type != VT_Object)
			return NULL;

		if(_value._map == NULL)
			return NULL;

		WTSParams* ret = static_cast<WTSParams*>(_value._map->get(name));
		return ret;
	}

	WTSParams* get(const std::string& name) const
	{
		if(_type != VT_Object)
			return NULL;

		if(_value._map == NULL)
			return NULL;

		WTSParams* ret = static_cast<WTSParams*>(_value._map->get(name));
		return ret;
	}

	WTSParams* get(uint32_t idx) const
	{
		if(_type != VT_Array)
			return NULL;

		if(_value._array == NULL)
			return NULL;

		WTSParams* ret = static_cast<WTSParams*>(_value._array->at(idx));
		return ret;
	}

	// 向Object类型对象的map中添加数据
	bool append(const char* _name, const char* _string, const char* desc="")
	{
		if(_type != VT_Object)
			return false;

		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSParams* item = WTSParams::create(_string, desc);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, int32_t _i32, const char* desc="")
	{
		if(_type != VT_Object)
			return false;
		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSParams* item = WTSParams::create(_i32, desc);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, uint32_t _u32, const char* desc="")
	{
		if(_type != VT_Object)
			return false;
		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSParams* item = WTSParams::create(_u32, desc);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, double _real, const char* desc="")
	{
		if(_type != VT_Object)
			return false;
		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSParams* item = WTSParams::create(_real, desc);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, bool _bool, const char* desc="")
	{
		if(_type != VT_Object)
			return false;
		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		WTSParams* item = WTSParams::create(_bool, desc);
		_value._map->add(_name, item, false);
		return true;
	}

	bool append(const char* _name, WTSParams *item, bool bAutoRetain = true)
	{
		if(_type != VT_Object || NULL == item)
			return false;
		if(_value._map == NULL)
		{
			_value._map = ChildrenMap::create();
		}
		_value._map->add(_name, item, bAutoRetain);
		return true;
	}

	// 向Array类型对象的array中添加数据
	bool append(int32_t _i32, const char* desc="")
	{
		if(_type != VT_Array )
			return false;
		if(_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSParams* item = WTSParams::create(_i32, desc);
		_value._array->append(item, false);
		return true;
	}

	bool append(uint32_t _u32, const char* desc="")
	{
		if(_type != VT_Array )
			return false;
		if(_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSParams* item = WTSParams::create(_u32, desc);
		_value._array->append(item, false);
		return true;
	}

	bool append(double _real, const char* desc="")
	{
		if(_type != VT_Array )
			return false;
		if(_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSParams* item = WTSParams::create(_real, desc);
		_value._array->append(item, false);
		return true;
	}

	bool append(bool _bool, const char* desc="")
	{
		if(_type != VT_Array )
			return false;
		if(_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		WTSParams* item = WTSParams::create(_bool, desc);
		_value._array->append(item, false);
		return true;
	}

	bool append( WTSParams *item, bool bAutoRetain = true)
	{
		if(_type != VT_Array || NULL == item)
			return false;
		if(_value._array == NULL)
		{
			_value._array = ChildrenArray::create();
		}
		_value._array->append(item, bAutoRetain);
		return true;
	}

	// 获取数据大小
	uint32_t size() const
	{
		if(_type == VT_Value)
			return 0;
		else if(_type == VT_Array)
		{
			return (_value._array == NULL)?0:_value._array->size();
		}
		else
		{
			return (_value._map == NULL)?0:_value._map->size();
		}
	}

	// 获取Object对象的所有键名
	MemberNames memberNames() const
	{
		MemberNames names;
		if(_type == VT_Object && _value._map != NULL)
		{
			auto it = _value._map->begin();
			for( ; it != _value._map->end(); it++)
			{
				names.emplace_back(it->first);
			}
		}
		return names;
	}

	// 释放内存
	virtual void release()
	{
		if(isSingleRefs())
		{
			switch(_type)
			{
			case VT_Array:
				if(NULL != _value._array)
				{
					_value._array->release();
				}
				break;
			case VT_Object:
				if(NULL != _value._map)
				{
					_value._map->release();
				}
				break;
			case VT_Value:
				if(NULL != _value._string)
				{
					delete _value._string;
				}
				break;
			}
		}
		WTSObject::release();
	}

	// 设置/获取描述信息
	inline void setDescription(const char* desc){_desc = desc;}
	inline const char* description() const{return _desc.c_str();}
};
```