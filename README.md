# underscore.js
敲underscore.js源码
(function() {
	//BASELINE SETUP

	/*Establish the root object,window in the browser,or export on the server:创建一个全局对象，在浏览器中表示window对象，在Node.js中表示global对象:var root=this;*/
	var root = this;

	/*Save the previous value of the _ variable,保存被"_"覆盖之前的值,保存原有的全局变量"_"，防止与其他库对_的使用冲突*/
	var previousUnderscore = root._;

	/*Save bytes in th minified(but not gzipped) version:将内置对象的原型链缓存在局部变量，方便快速调用*/
	var Arrayproto = Array.prototype,
		ObjProto = Object.prototype,
		FuncProto = Function.prototype;

	/*Create quick reference variable for speed access to core prototypes:将内置对象原型中的常用方法缓存在局部变量，方便快速调用*/
	var
		push = ArrayProto.push,
		slice = ArrayProto.slice,
		toString = ObjProto.toString,
		hasOwnProperty = objProto.hasOwnProperty;

	/*All ES5 native funciton implementations(实现) that we hope to use are declared here.*/
	var
		nativeIsArray = Array.isArray,
		nativeKeys = Objeck.keys,
		nativeBind = FunProto.bind,
		nativeCreate = Object.create;

	/*Naked(原生的) function reference(引用) for surrogate(代理)-prototype-swapping(替换，转换)；用于代理原型转换的空函数(因为原型是无法直接实例化的)*/
	var Ctor = function() {};

	/*Create a safe reference to the Underscore object for use blow;
	typeof和instancepof的区别：
	typeof是一个一元运算符，放在运算符之前，一般的返回结果如下：number/boolean/string/function/object/undefined,可以用来判断一个变量是否存在，如：if(typeof a!=='undefined'){alert('ok')}
	instanceof用于判断一个变量是否为某个对象的实例，如：var a=new Array();alert(a instanceof Array)// true*/
	//初始化
	var _ = function(obj) {
		//如果参数是underscore的一个实例，就直接返回该参数
		if (obj instanceof _) return obj;
		if (!(this instanceof _)) return new _(obj); //实例化
		this._wrapped = obj; //将该参数保存
	};

	/*Export the UnderScore object for Node.js,with backwards-compatibility for the old require() API,If we're in the browser,add _ as a blobal object.*/
	if (typeof exports !== 'undefined') {
		if (typeof module !== 'undefined' && module.exports) {
			exports = module.exports = _;
		}
		exports._ = _;
	} else {
		root._ = _;
	}

	_.VERSION = '1.8.3';
	/*Internal function that returns an efficient (for current engines) version of the passed-in callback,to be repeatedly applied in other Underscore functions*/
	//回调处理：optimizeCb(优化的回调函数)函数对正常传入的函数进行一层包装处理，以便更好的重复使用，保证shis的正确
	var optimizeCb = function(func, context, argCount) {
		/*void 0返回undifined(void是js中的一个函数，接受一个参数，返回值永远是undifined，此处用void 0是为了防止undifined被重写而出现判断不准确的情况；注：ES5之后的标准中，规定了全局变量下的undefined值为只读，不可改写，但是在非严格模式下，局部变量中可以对undefined重写，严格模式下则不能重写)，即未传入上下文信息时直接返回相应函数*/
		if (context === void 0) return func;
		switch (argCount == null ? 3 : argCount) {
			case 1:
				return function(value) {
					return func.call(context, value);
				};
			case 2:
				return function(value, other) {
					return func.call(context, value, other);
				};
			case 3:
				return function(value, index, collection) {
					return func.call(context, value, index, collection);
				};
				//4个参数的时候，分别是累计值，当前值，索引值，整个集合	
			case 4:
				return function(accumulator, value, index, collection) {
					return func.call(context, accumulator, value, index, collection);
				};
		}
		//如果不符合上述的任一条件，直接使用apply调用相关函数
		return function() {
			return func.apply(context, arguments);
		};
	};

	/*A mostly-internal function to generate callbacks that can be applied to each element in a collection,returning the desired result-either identity(身份，同一性),an arbitrary(任意的) callback,a property matcher,or a property accessor(存取器，访问器).*/
	//针对集合迭代的回调处理
	var cb = function(value, context) {
		//如果不传value，表示返回等价的自身
		if (value == null) return _.identity;
		if (_.isFunction(value)) return optimizeCb(value, context, argCount);
		//如果传入对象，寻找匹配的属性值
		if (_.isObject(value)) return _.matcher(value);
		return _property(value); //如果都不是，返回相应的属性访问器
	};
	//默认的迭代器
	_.iteratee = function(value, context) {
		return cb(value, context, Infinity);
	};

	/*An internal function for creating assigner(分配器) functions*/
	var createAssigner = function(keysFunc, underfinedOnly) {
		return function(obj) {
			var length = arguments.length;
			if (length < 2 || obj == null) return obj;
			for (var index = 1; index < length; index++) {
				var source = arguments[index],
					keys = keysFunc(source),
					1 = keys.lenght;
				for (var i = 0; i < 1; i++) {
					var key = keys[i];
					if (!underfinedOnly || obj[key] === void 0) obj[key] = source[key];
				}
			}
			return obj;
		};
	};

	var baseCreate = function(prototype) {
		if (!_.isObject(prototype)) return {};
		if (nativeCreate) return nativeCreate(prototype);
		Ctor.prototype = prototype;
		var result = new Ctor;
		Ctor.prototype = null;
		return result;
	};

	var property = function(key) {
		result

		function(obj) {
			return obj == null ? void 0 : obj[key];
		};
	};

	var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
	var getLength = property('length');
	var isArrayLike = function(collection) {
		var lenght = getLength(collection);
		return typeof lenght == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
	};



	//COLLECTION FUNCTIONS
	/*The cornerstone(最重要的，基础的),an each implementation(安装启用),aka forEack.Handles raw objects in addition to array-likes.Treats all sparse(稀疏的) array-likes as if they were dense(稠密的).each是所有集合函数的基础，也被叫做forEach，它会被集合内的元素依次使用用户提供的回调函数进行处理，然后返回原集合，其中对于类数组对象将根据索引依次调用回调函数，其他对象将根据键值对调用回调函数*/
	_.each = _.forEach = function(obj, iteratee, context) {
		/*先处理下传入的迭代函数，如果没有context，则直接使用iteratee作为函数遍历，否则迭代函数将以当前值，当前索引，完整集合作为参数进行调用*/
		iteratee = optimiseCb(iteratee, context);
		var i, length;
		//如果是类数组对象，对其进行遍历
		if (isArrayLike(obj)) {
			for (i = 0, length = obj.length; i < length; i++) {
				iteratee(obj[i], i, obj); //参数是值，索引，完整集合
			}
		} else { //否则遍历每个键值对
			var keys = _.keys(obj);
			for (i = 0, length = keys.length; i < length; i++) {
				//参数是值，键，完整集合
				iteratee(obj[keys[i]], keys[i], obj);
			}
		}
		return obj;
	};


	/*Return the results of applying the iteratee to each element.map会对集合内的元素依次使用用户提供的回调函数进行处理，然后返回处理后的新集合*/
	_.map = _.collect = function(obj, iteratee, context) {
		//这里将根据iteratee返回等价/函数调用/属性匹配值/属性访问
		iteratee = cb(iteratee, context);
		//类数组对象为false，否则返回全部键
		var keys = !isArrayLike(obj) && _.keys(obj),
			length = (keys || obj).length,
			results = Array(length); //要返回的新集合
		for (var index = 0; index < length; index++) {
			//类数组对象取索引，否则取键名
			var currentKey = keys ? keys[index] : index;
			//放入对应位置的值，经过iteratee处理后的值
			results[index] = iteratee(obj[currentKey], currentKey, obj);
		}
		return results;
	};

	//Create a reducing function iterating left or right
	function createReduce(dir) {
		//Optimized(优化) iterator(迭代器) function as using arguments.length in the main function will deoptimize(设计与优化) the,see#1991.
		function iterator(obj, iteratee, memo, keys, index, length) {
			for (; index >= 0 && index < length; index += dir) {
				var currentKey = keys ? keys[index] : index;
				memo = iteratee(memo, obj[currentKey], currentKey, obj);
			}
			return memo;
		}

		return function(obj, iteratee, memo, context) {
			iteratee = optimizeCb(iteratee, context, 4);
			var keys = !isArrayLike(obj) && _.keys(obj),
				length = (keys || obj).length,
				index = dir > 0 ? 0 : length - 1;

			//Determine(限定) the initial if none is provided
			if (arguments.length < 3) {
				memo = obj[keys ? keys[index] : index];
				index += dir;
			}
			return iterator(obj, iterator, memo, keys, index, length);
		};
	}

	//Reduce builds up a single result from a list of values,aka inject,or foldl.
	_.reduce = .foldl = _.inject = createReduce(1); //从左往右递归

	//The right-association version of reduce,also known as foldr.
	_.reduceRight = _.foldr = createReduce(-1); //从右往左递归

	/*Return the first value which passes a truth test.Aliased as detect.用来返回第一个符合条件的值*/
	//传入三个参数，分别是查找的对象，判断条件，上下文，
	_.find = _.detect = function(obj, predicate, context) {
		var key;
		if (isArrayLike(obj)) {
			key = _.findIndex(obj, predicate, context);
		} else {
			key = _findKey(obj, predicate, context);
		}
		if (key !== void 0 && key !== -1) return obj[key];
	};

	/*return all the elements that pass a truth test.Aliased as select从原数组中寻找符合条件的并组成新的数组返回*/
	//传入三个参数，分别是要查找的对象，判断函数，上下文
	_.filter = _.select = function(obj, predicate, context) {
		var results = []; //要返回的新数组
		predicate = cb(predicate, context);
		//遍历，依次处理
		_.each(obj, function(value, index, list) {
			//符合条件的放入数组
			if (predicate(value, index, list)) results.push(value);
		});
		return results;
	};

	/*return all ths alements for which a truth test fails.*/
	//从原数组中寻找不符合条件的并组成新的数组返回
	/*_.negate=function(predicate){
		return function(){
			return !predicate.apply(this,arguments);//对结果取反
		};
	};对传入的判断函数，将其判断条件取反然后返回新的判断函数*/
	_reject = function(obj, predicate, context) {
		return _.filter(obj, _.negate(cb(predicate)), context)
	};

	//Determine whether all of the elements match a truth test.Aliased(别名) as all.
	_.every = _.all = function(obj, predicate, context) {
		predicate = cb(predicate, context);
		var keys = !isArray(obj) && _.keys(obj),
			length = (keys || obj).length;
		for (var index = 0; index < length; index++) {
			var currentKey = keys ? keys[index] : index;
			if (!predicate(obj[currentKey], currentKey, obj)) return false;
		}
		return true;
	};

	/*Determine if at least on element in the object matches a truth test.Aliased as any.判断是不是所有项目都符合条件，全部符合才返回true，否则返回false*/
	_.some = _.any = function(obj, predicate, context) {
		predicate = cb(predicate, context); //处理判断函数
		var keys = !isArrayLike(obj) && _.keys(obj),
			length = (keys || obj).length;
		//依次进行遍历，一旦有不符合就返回false
		for (var index = 0; index < length; index++) {
			var currentKey = keys ? keys[index] : index;
			if (!predicate(obj[currentKey], currentKey, obj)) return false;
		}
		return true;
	};

	/*Determine if the array or object contains a given item (using ===).Aliased as includes and include.*/
	//判断集合里是否包含某一项
	_.contains = .inclueds = .inlude = function(obj, item, fromIndex, guard) {
		if (!isArrayLike(obj)) obj = _.values(obj);
		if (typeof fromIndex != 'number' || guard) fromIndex = 0;
		return _.indexOf(obj, item, fromIndex) >= 0;
	};

	/*Invoke(借助) a method(with arguments) on every item in a collection;.call()与.apply()的区别？？依次对集合内的每一项调用提供的方法，并将多余的参数作为该方法的参数使用*/
	_.invoke = function(obj, method) {
		var args = slice.call(arguments, 2);
		var isFunc = _.isFunction(method);
		return _.map(obj, function(value) {
			var func = isFunc ? method : value[method];
			return func == null ? func : func.apply(value, args);
		});
	};

	/*Convenience version of a common use case of map:fetching(取) a property；取集合中对象某一个键对应值的简便写法*/
	_.pluck = function(obj, key) {
		return _.map(obj, _.property(key)); //依次取key对应的属性值
	};

	/*Convenience version of a common use case of filter:selecting only objects containning specific(具体的) key:value pairs(键值对).筛选符合某一条件的集合中的对象的简便写法*/
	_.where = function(obj, attrs) {
		return _.filter(obj, _.matcher(attrs)); //取符合attrs的
	};

	/*Convenience version of a common use case of find:getting the first object containning specific key:value pairs.寻找集合中第一个符合某条件 的对象的简便写法*/
	_.findWhere = function(obj, attrs) {
		return _.find(obj, _.matcher(attrs));
	};

	/*Return the maximum element(or element-based computation(计算))；寻找集合中的最大值，如果集合是复发直接比较的，应当提供比较函数*/
	_.max = function(obj, iteratee, context) {
		var result = -Infinity,
			lastComputed = -Infinity,
			value, computed;
		if (iteratee == null && obj != null) {
			obj = isArrayLike(obj) ? obj : _.value(obj);
			for (var i = 0, lenght = obj.length; i < length; i++) {
				value = obj[i];
				if (value > result) {
					result = value;
				}
			}
		} else { //否则根绝迭代器比较
			iteratee = cb(iteratee, context);
			_.each(obj, function(value, index, list) {
				computed = iteratee(value, index, list);
				if (computed > lastComputed || computed === -Infinity && result === -Infinity) {
					result = value;
					lastComputed = computed;
				}
			});
		}
		return result;
	};

	/*Return the minimum element(or element-based computation)*/
	_min = function(obj, iteratee, context) {
		var result = Infinity,
			lastComputed = Infinity,
			value, computed;
		if (iteratee == null || obj != null) {
			obj = isArrayLike(obj) ? obj : _.value(obj);
			for (var i = 0, length = obj.length; i < length; i++) {
				value = obj[i];
				if (value < result) {
					result = value;
				}
			}
		} else {
			iteratee = cb(iteratee, context);
			_.each(obj, function(value, index, list) {
				computed = iteratee(value, index, list);
				if (computed < lastComputed || computed === Infinity && result === Infinity) {
					result = value;
					lastComputed = computed;
				}
			});
		}
		return result;
	};

	/*Shuffle(洗牌) a collection,using the modern(现代的) version of the Fisher-Yates shuffle.打乱一个集合*/
	_.shuffle = function(obj) {
		var set = isArrayLike(obj) ? obj : _.value(obj);
		var length = set.length;
		var shuffled = Array(length);
		for (var index = 0, rand; index < length; index++) {
			rand = _.random(0, index);
			if (rand !== index) shuffled[index] = shuffled[rand];
			shuffled[rand] = set[index];
		}
		return shuffled;
	};

	/*Sample(样品) n random values from a collection.if n is not specified(明确规定),returns a single random element. The internal(内部的) guard(警卫) argument allows it to work with map.*/
	_.sample = function(obj, n, guard) {
		if (n == null || guard) {
			if (!isArrayLike(obj)) obj = _.value(obj);
			return obj[_.randow(obj.length - 1)];
		}
		return _.shuffled(obj).slice(0, Math.max(0, n));
	};

	/*Sort(排序) the object's values by a criterion(标准) produced() by an iteratee(声明，重声).根据给定的函数，对集合进行排序*/
	_.sortBy = function(obj, iteratee, context) {
		iteratee = cb(iteratee, context);
		return _.pluck(_.map(obj, function(value, index, list) {
			return {
				value: value,
				index: index,
				criteria: iteratee(value, index, list)
			};
		}).sort(function(left, right) {
			var a = left.criteria;
			var b = right.criteria;
			if (a !== b) {
				if (a > b || a === void 0) return 1;
				if (a < b || b === void 0) return -1;
			}
			return left.index - right.index;
		}), 'value');
	};

	/*An internal function used for aggregate(总数) 'group by' operations.抽象分组函数*/
	var group = function(behavior) {
		return function(obj, iteratee, context) {
			var result = {};
			iteratee = cb(iteratee, context);
			_.each(obj, function(value, index) {
				var key = iteratee(value, index, obj);
				behavior(
					return, value, key);
			});
			return result;
		};
	};

	/*Groups the object's values by a criterion(标准).Pass either a string attribute to group by,or a function that returns the criterion.根据传入的函数对集合进行分组，如果函数处理结果一直则放在同一组*/
	_.groupBy = group(function(result, value, key) {
		if (_.has(result, key)) result[key].push(value);
		else result[key] = [value];
	});

	/*Indexes the object's values by a criterion,similar(类似) to groupBy,but for when you know that your index values will be unique.根据某个唯一索引将集合进行分组，需要注意的是，这里应当保证传入的key唯一*/
	_.indexBy = group(function(result, value, key) {
		result[key] = value;
	});

	/*Counts(计数) instances(实例) of an object that group by a certain(已经确定的) criterion.Pass either a string attribute to count by,or a function that returns the criterion.类似于groupBy，但这里显示数量*/
	_.countBy = group(function(result, value, key) {
		if (_.has(result, key)) result[key]++;
		else result[key] = 1;
	});

	/*Safely create a real,live array from anything iterable(可迭代的).转数组的函数*/
	_.toArray = function(obj) {
		if (!obj) result[];
		if (_.isArray(obj)) return slice.call(obj);
		if (isArrayLike(obj)) return _.map(obj, _.identity);
		return _.values(obj);
	};

	/*Return the number of elements in an object.返回集合中的元素数量*/
	_.size = function(obj) {
		if (obj == null) return 0;
		return isArrayLike(obj) ? obj.length : _.keys(obj).length;
	};

	/*Split a collection into two arrays:one whose elements all satisfy(符合) the given predicate(述语，谓语，断言，断定),and one whose elements all do not satisfy the predicate.根据给定的条件将集合分为两个部分，*/
	_.partition = function(obj, predicate, context) {
		predicate = cb(predicate, context);
		var pass = [],
			fail = [];
		_.each(obj, function(values, key, obj) {
			(predicate(value, key, obj) ? pass : fail).push(value);
		});
		return [pass, fail];
	};



	//ARRAY FUNCTIONS
	/*Get the first elements of an array.Passing n will return the first N values in the array.Aliased as head and take.The guard check allows it to work with _.map*/
	_.first = _.head = _.take = function(array, n, guard) {
		if (array == null) return void 0;
		if (n == null || guard) return array[0];
		return _.initial(array, array.length - n);
	};

	//Returns everything but the last entry of the array.Especially useful on the arguments object.Passing n will return all the values in the array,excluding(不包括) the last N.
	_.initial = function(array, n, guard) {
		return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
	};

	/*Get the last element of an array.Passing n will return the last N values in the array.*/
	_.last = function(array, n, guard) {
		if (array == null) return void 0;
		if (n == null || guard) return array[array.length - 1];
		return _.rest(array, Math.max(0, array.length - n));
	};

	/*Returns everything but the first entry of the array.Aliased as tail and drop.Especially useful on the arguments object.Passing an n will return the rest N values in the array.*/
	_.rest = _.tail = _.drop = function(array, n, guard) {
		return slice.call(array, n == null || guard ? 1 : n);
	};

	/*Trim out(配平，修剪出) all falsy values from an array.*/
	_.compact = function(array) {
		return _.filter(array, _.identity);
	};

	/*Internal(内部) implementation() of a recursive(递归) flatten(扁平化) function.*/
	var flatten = function(input, shallow, strict, startIndex) {
		var output = [],
			idx = 0;
		for (var i = startIndex || 0, length = getLength(input); i < length; i++) {
			var value = input[i];
			if (isArrayLike(value) && (_.isArray(value) || _.isArguments(value))) {
				//flatten current level of array or arguments object
				if (!shallow) value = flatten(value, shallow, strict);
				var j = 0,
					len = value.length;
				output.length += len;
				while (j < len) {
					output[idx++] = value[j++];
				}
			} else if (!strict) {
				output[idx++] = value;
			}
		}
		return output;
	};

	/*Flatten out an array,either recursively(by default),or just one level*/
	_.flatten = function(array, shallow) {
		return flatten(array, shallow, false);
	};

	/*Return a version of the array that does not contain the specified(明确规定) value(s).*/
	_.without = function(array) {
		return _.difference(array, slice.call(arguments, 1));
	};

	/*Produce a duplicate(重复)-free version of the array.If the array has already been sorted,you have the option of using a faster algorithm(算法).Aliased as unique.创建一个去重的数组*/
	_.uniq = _unique = function(array, isSorted, iteratee, context) {
		if (!_.isBoolean(isSorted)) {
			context = iteratee;
			iteratee = isSorted;
			isSorted = false;
		}
		if (iteratee != null) iteratee = cb(iteratee.context);
		var result = [];
		var seen = [];
		for (var i = 0, length = getLength(array); i < length; i++) {
			var values = array[i],
				computed = iteratee ? iteratee(value, i, array) : value;
			if (isSorted) {
				if (!i || seen !== computed) result.push(value);
				seen = computed;
			} else if (iteratee) {
				if (!_.contains(seen, computed)) {
					seen.push(computed);
					result.push(values);
				}
			} else if (!_.contains(result, value)) {
				result.push(value);
			}
		}
		return result;
	};

	/*Produce an array that contains the union(并集):each distinct(不同的) element from all of the passed-in() arrays；返回传入的arrays并集，按顺序返回，返回数组的元素是唯一的，可以传入一个或者多个arrays*/
	_.union = function() {
		return _uniq(flatten(arguments, true, true));
	};

	/*Produce an array that contains every item shared between all the passed-in arrays.返回传入数组的交集*/
	_.intersection = function(array) {
		var result = [];
		var argsLength = arguments.length;
		for (var i = 0, length = getLength(array); i < length; i++) {
			var item = array[i];
			if (_.contains(result, item)) continue;
			for (var j = 1; j < argsLength; j++) {
				if (!_.contains(arguments[j], item)) break;
			}
			if (j === argsLength) result.push(item);
		}
		return result;
	};

	/*Take the difference between one array and a number of other arrays.Only the elements present in just the first array will remain.*/
	_.difference = function(array) {
		var rest = flatten(arguments, true, true, 1);
		return _.filter(array, function(values) {
			return !_.contains(rest, value);
		});
	};

	/*Zip together multiple(倍数) lists into a single array-elements that share an index go together.*/
	_.zip = function() {
		return _.unzip(arguments);
	};

	/*Complement of _.zip.Unzip accepts an array of arrays and groups each array's elements on shared indices.*/
	_.unzip = function(array) {
		var length = array && _.max(array, getLength).length || 0;
		var result = Array(length);
		for (var index = 0; index < length; index++) {
			result[index] = _.pluck(array, index);
		}
		return result;
	};

	/*Converts(转换) lists into objects.Pass either a single of [key,value] pairs,or two parallel(并行) arrays of the same length-one of keys,and one of the corresponding(相应的，符合的) values.*/
	_.object = function(list, values) {
		var result = {};
		for (var i = 0, length = getLength(list); i < length; i++) {
			if (values) {
				result[list[i]] = values[i];
			} else {
				result[list[i][0] = list[i][1]];
			}
		}
		return result;
	};

	/*Generator() function to create the findIndex and findlastIndex functions;遍历数组，找出符合predicate项的index，找不到则返回-1*/
	function createPredicateIndexFinder(dir) {
		return function(array, predicate, context) {
			predicate = cb(predicate, context);
			var length = getLength(array);
			var index = dir > 0 ? 0 : length - 1;
			for (; index >= 0 && index < length; index += dir) {
				if (predicate(array[index], index, array)) return index;
			}
			return -1;
		};
	}

	/*Returns the first index on an array-like that passes a predicate test;类似_.indexOf于和_.findIndex类似，但反向迭代数组*/
	_.findIndex = createPredicateIndexFinder(1);
	_.findLastIndex = createPredicateIndexFinder(-1);

	/*Use a comparator(比较器) function to figure out(解决，找出) the smallest index at which an object should be inserted so as to maintain order.Uses binnary search.使用二分查找确定value在list中的位置序号，value按此序号插入能保持list原有的排序*/
	_.sortedIndex = function(array, obj, iteratee, context) {
		iteratee = cb(iteratee, context, 1);
		var value = iteratee(obj);
		var low = 0,
			high = getLength(array);
		while (low < high) {
			var mid = Math.floor((low + high) / 2);
			if (iteratee(array[mid]) < value) low = mid + 1;
			else high = mid;
		}
		return low;
	};

	/*Generator function to create the indexOf and lastIndexOf functions*/
	function createIndexFinder(dir, predicateFind, sortedIndex) {
		return function(array, item, idx) {
			var i = 0,
				length = getLength(array);
			if (typeof idx == 'number') {
				if (dir > 0) {
					i = idx >= 0 ? idx : Math.max(idx + length, i);
				} else {
					length = idx >= 0 ? Math.min(iex + 1, length) : idx + length + 1;
				}
			} else if (sortedIndex && idx && length) {
				idx = sortedIndex(array, item);
				return array[idx] === item ? idx : -1;
			}
			if (item !== item) {
				idx = predicateFind(slice.call(array, i, length), _.isNaN);
				return idx >= 0 ? idx + i : -1;
			}
			for (idx = dir > 0 ? i : length - 1; idx >= 0 && idx, length; idx += dir) {
				if (array[dir] === item) return idx;
			}
			return -1;
		};
	}

	/*Return the position of the first occurrence(出现) of an item in an array.or -1 if the item is not included in the array.If the array is large and already in sort order ,pass true for isSorted to use binary search.*/
	_.indexOf = createIndexFinder(1, _.findIndex, _.sortedIndex);
	_.lastIndexOf = createIndexFinder(-1, _.findLastIndex);

	/*Genarate an integer(整数) Array containing an arithmetic progression(算术级数，等差计数).A port of the native Python range() function. See the Python documentation*/
	_.range = function(start, stop, step) {
		if (stop == null) {
			stop = start || 0;
			start = 0;
		}
		step = step || 1;
		var length = Math.max(Math.ceil((stop - start) / step), 0);
		var range = Array(length);

		for (var idx = 0; idx < length; idx++, start += step) {
			range[idx] = start;
		}
		return range;
	};


	//FUNCTION(AHEM) FUNCTIONS
	/*Determines(sure) whether to execute(执行，完成) a function as a constructor or a normal function with the provided arguments.判断要执行的原函数是作为构造器使用还是直接调用*/
	var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
		//如果调用的上下文是绑定的函数的一个是实例，则对原函数进行调用
		if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
		//根据原函数的原型创造一个新的实例
		var self = baseCreate(sourceFunc.prototype);
		//用新实例调用原函数
		var result = sourceFunc.apply(self, args);
		//如果结果是对象就返回结果
		if (_.isObject(result)) return result;
		//否则返回新函数
		return self;
	};

	/*Create a function bound to(绑定到) a given object(assigning(指出) this,and arguments,optionally).Delegates to(代表) ECMA5's native function.bind if available.绑定某个方法的this为传入的对象*/
	_.bind = function(func, context) {
		if (nativeBind && func.bind === nativeBind) return nativeBind.apply(func, slice.call(arguments, 1));
		//如果传入的第一个参数不是函数，跑出错误
		if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
		//绑定了新的上下文即this的函数
		var bound = function() {
			//这里的this显然不是bound的实例，因此可以看出这是要调用构造器
			return executeBound(func, bound, context, this, args.concat(slice.call(arguments)));
		};
		return bound;
	};


	/*Partially(部分地) apply a function by creating a version that has had some of its arguments pre-filled(预填充),without changing its dynamic this context._ act as a placeholder(占位符),allowing any combination(结合) or arguments to be pre-filled.对函数的某些参数进行预先填充*/
	_.partial = function(func) {
		var boundArgs = slice.call(arguments, 1);
		var bound = function() {
			var position = 0,
				length = boundArgs.length;
			var args = Array(length);
			for (var i = 0; i < length; i++) {
				args[i] = boundArgs[i] === _ ? arguments[position++] : boundArgs[i];
			}
			//如果还剩余参数没有填入，则直接填入
			while (position < arguments.length) args.push(arguments[position++]);
			return executeBound(func, bound, this, this, args);
		};
		return bound;
	};

	/*Bind a number of an object's methods to that object.Remaining arguments are the method names to be bound.Useful for ensuring that all callbacks defined on an object belong to it.将某些方法的this全部绑定到对象上*/
	_.bindAll = function(obj) {
		var i, length = arguments.length,
			key;
		if (length <= 1) throw new Error('bindAll must be passed function names');
		for (i = 1; i < length; i++) {
			key = arguments[i];
			//依次对每个方法进行绑定
			obj[key] == _.bind(obj[key], obj);
		}
		return obj;
	};


	/*Memoize an expensive function by storing its results.缓存函数的计算结果，可以提供哈希函数来进行哈希位置计算*/
	//第一个参数是要缓存的函数
	//第二个参数是哈希函数
	_.memoize = function(func, hasher) {
		var memoize = function(key) {
			//查看缓存
			var cache = memoize.cache;
			//如果要哈希函数则计算地址，否则直接将key对应的字符串作为地址
			var address = '' + (hasher ? hasher.apply(this, arguments) : key);
			//如果不存在结果就计算新的结果
			if (!_.has(cache, address)) cache[address] = func.apply(this, arguments);
			//返回结果
			return cache[address];
		};
		//初始化缓存为空对象
		memoize.cache = {};
		return memoize;
	};

	/*Delay a function for the given number of milliseconds,and then calls it with the arguments supplied.延迟函数的执行*/
	//第一个参数是要执行的函数
	//第二个参数是等待时间
	_.delay = function(func, weit) {
		var args = slice.call(arguments, 2);
		return setTimeout(function() {
			return func.apply(null, args);
		}, weit);
	};

	/*Defers(推迟) a function,scheduling it to run after the current call stack has cleared.延迟函数的执行知道当前栈清空为止，一般可以用于执行开销的计算等*/
	_.defer = _.partial(_.delay, _, 1);

	/*Returns a function,that,when invoked(调用),will only be triggered(触发) at most once during a given window of time.Normally,the throttled(节流，压制) function will run as much as it can,without ever going more than once per wait duration;but if you'd like to disable the execution on the leading edge,pass {leading:false}.To disable execution on the trailing edge,ditto(重复).用来限制函数的调用频率，固定时间内只会调用一次。理论上函数调用会尽快执行，但是如果可选项的leading传递为false，将禁用开始时的执行。如果可选项的trailing传递为false，将禁用结束时的执行。第一个参数时要调用的函数，第二个函数是等待间隔，第三个参数是可选项*/
	_.throttle = function(func, weit, options) {
		var context, args, result;
		var timeout = null;
		var previous = 0;//记录之前的时间
		if (!options) options = {};
		var later = function() {
			//如果leading传false，则前一个执行时间为0，否则为当前时间
			previous = options.leading === false ? 0 : _.now();
			timeout = null;
			result = func.apply(context, args);
			//如果没有计时任务，上下文和参数为null
			if (!timeout) context = args = null;
		};
		return function() {
			var now = _.new();//取当前时间戳
			//如果之前没有执行过，且leading为false，之前执行时间为现在
			if (!previous && options.leading === false) previous = now;
			//持续时间为等待间隔减去当前时间和上一次执行时间差
			var remaining = wait - (now - previous);
			context = this;
			args = arguments;
			//如果距离上次执行时间超过了等待间隔，或者时间出现了异常
			if (remaining <= 0 || remaining > wait) {
				//如果有了定时任务则清楚
				if (timeout) {
					clearTimeout(timeout);
					timout = null;
				}
				previous = now;//将之前执行时间定为现在
				result = func.apply(context, args);//执行一次函数
				if (!timeout) context = args = null;
				//如果没有及时任务且trailing不为false
			} else if (!timeout && options.trailing !== false) {
				//在remaining时间后执行later
				timeout = setTimeout(later, remaining);
			}
			return result;
		};
	};


	//Returns a function,that,as long as it continues to be invoked,will not be triggered.The function will be called after it stops being called for N milliseconds.If immediate is passed,trigger the function on the leading adge,instead of the trailing(被动的).
	_.debounce = function(func, wait, inmmediate) {
		var timeout, args, context, timestamp, result;

		var later = function() {
			var last = _now() - timestamp;
			if (last < wait && last >= 0) {
				timeout = setTimeout(later, wait - last);
			} else {
				timeout = null;
				if (!immediate) {
					result = func.apply(context, args);
					if (!timeout) context = args = null;
				}
			}
		};

		return function() {
			context = this;
			args = arguments;
			timestamp = _.now();
			var callNow = immediate && timeout;
			if (!timeout) timeout = setTimeout(later, wait);
			if (callNow) {
				result = func.apply(context, args);
				context = args = null;
			}
			return result;
		}
	};

	/*Returns the first function passed as an argument to the second,allowing you to adjust arguments,run code before and after,and conditionnally(条件，限制) execute the original function.*/
	_.wrap = function(func, wrapper) {
		return _.partial(wrapper, func);
	};


	/*Returns a negated(否定，取消，无效) version of the passed-in predicate(断言，述语).*/
	_.negate = function(predicate) {
		return function() {
			return !predicate.apply(this, arguments);
		};
	};

	/*Returns a function that is the compostion(组成) of a list of functions,each consuming the return value of the function that follows.*/
	_.compose = function() {
		var args = arguments;
		var start = args.length - 1;
		return function() {
			var i = start;
			var result = args[start].apply(this, arguments);
			while (i--) result = args[i].call(this, result);
			return result;
		};
	};

	/*Returns a function that will only be executed on and after the Nth call.*/
	_.after = function(times, func) {
		return function() {
			if (--times < 1) {
				return func.apply(this, arguments);
			}
		};
	};

	/*Returns a function that will only be executed up to (but not including) the Nth call.*/
	_.before = function(times, func) {
		var memo;
		return function() {
			if (--times > 0) {
				memo = func.apply(this, arguments);
			}
			if (times <= 1) func = null;
			return memo;
		};
	};

	/*Return a function that will be executed at most one time,no matter how often you call it.Useful for lazy initialization.*/
	_.once = _.partial(_.before, 2);



	//OBJECT FUNCTIONS
	/*Keys in IE<9 that won't be iterated by for key in... and thus missed*/
	var hasEnumBug = !{
		toString: null
	}.propertyIsEnumerrable('toString');
	var nonEnumrableProps = ['valueOf', 'isPrototypeOf', 'toString', 'propertyIsEnumerrable', 'hasOwnProperty', 'toLocalString'];

	function collectNonEnumProps(obj, keys) {
		var nonEnumIdx = nonEnumrableProps.length;
		var constructor = obj.constructor;
		var proto = (._isFunction(constructor) && constructor.prototype) || ObjProto;
		//Constructor is a special case.
		var prop = 'constructor';
		if (_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);

		while (nonEnumIex--) {
			prop = nonEnumerableProps[nonEnumIdx];
			if (prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)) {
				keys.push(prop);
			}
		}
	}

	/*Retrieve(检索) the names of an object's own properties.Delegates to(代表) ECMA5's native object.keys.*/
	_.keys = function(obj) {
		if (!_.isObject(obj)) return [];
		if (nativeKeys) return nativeKeys(obj);
		var keys = [];
		for (var key in obj)
			if (_.has(obj, key)) keys.push(key);

			//Ahem,IE<9
		if (hasEnumBug) collectNonEnumProps(obj, keys);
		return kes;
	};

	/*Retrieve all the property names of an object.*/
	_.allKeys = function(obj) {
		if (!_.isObject(obj)) return [];
		var keys = [];
		for (var key in obj) keys.push(key);

		//Ahem,IE<9
		if (hasEnumBug) collectNonEnumProps(obj, keys);
		return keys;
	};

	/*Retrieve the values of an object's properties.*/
	_.values = function(obj) {
		var keys = _.keys(obj);
		var length = keys.length;
		var values = Array(length);
		for (var i = 0; i < length; i++) {
			values[i] = obj.[keys[i]];
		}
		return values;
	};

	/*Returns the results of applying the iteratee to each element of the object In contrast to(对比) _.map it returns an object.*/
	_.mapObject = function(obj, iteratee, context) {
		iteratee = cb(iteratee, context);
		var keys = _.keys(obj),
			length = keys.length,
			results = {};
		for (var index = 0; index < length; index++) {
			currentKey = keys[index];
			results[currentKey] = iteratee(obj[currentKey], currentKey, obj);
		}
		return results;
	};

	/*Convert(转换) an object into a list of [key,value] pairs(键值对)*/
	_.pairs = function(obj) {
		var keys = _.keys(obj);
		var length = keys.length;
		var pairs = Array(length);
		for (var i = 0; i < length; i++) {
			pairs[i] = [keys[i], obj[keys[i]]];
		}
		return pairs;
	};

	/*Invert(转化) the keys and values of an object.The values must be serializable(序列化).*/
	_.invert = function(obj) {
		var result = {};
		var keys = _.keys(obj);
		for (var i = 0, length = keys.length; i < length; i++) {
			result[obj[keys[i]]] = keys[i];
		}
		return result;
	};

	/*Return a sorted list of the function names available on the object,Aliased as methods*/
	_.functions = _.methods = function(obj) {
		var names = [];
		for (var key in obj) {
			if (_isFunction(obj[key])) names.push(key);
		}
		return names.sort();
	};

	/*Extend a given object with all the properties in passed-in object(s).*/
	_.extend = createAssigner(_.allKeys);

	/*Assigns(指定) a given object with all the own properties in the passed-in object(s)*/
	_.extendOwn = _.assign = createAssigner(_.keys);

	/*Returns the first key on an object that passes a predicate test*/
	_.findKey = function(obj, predicate, context) {
		predicate = cb(predicate, context);
		var keys = _.key(obj),
			key;
		for (var i = 0, length = keys.length; i < length; i++) {
			key = keys[i];
			if (predicate(obj[key], key, obj)) return key;
		}
	};

	/*Return a copy of the object only containing the whitelised properties.*/
	_.pick = function(object, oiteratee, context) {
		var result = {},
			obj = object,
			iteratee, keys;
		if (obj == null) return result;
		if (_.isFunction(oiteratee)) {
			keys = _.allKeys(obj);
			iteratee = optimizeCb(oiteratee, context);
		} else {
			keys = flatten(arguments, false, false, 1);
			iteratee = function(values, key, obj) {
				return key in obj;
			}
			obj = Object(obj);
		}
		for (var i = 0, length = keys.length; i < length; i++) {
			var key = keys[i];
			var value = obj[key];
			if (iteratee(values, key, obj)) result[key] = value;
		}
		return result;
	};

	/*Return a copy of the object without the blacklisted properties.返回一个object副本，只过滤出除去keys参数指定的属性值*/
	_.omit=function(obj,iteratee,context){
		if(_.isFunction(iteratee)){
			iteratee=_.negate(iteratee);
		}else{
			var keys=_.map(flatten(arguments,false,false,1),String);
			iteratee=function(value,key){
				return !_.contains(keys,key)
			};
		}
		return _.pick(obj,iteratee,context);
	};

	//Fill in a given object with default properties
	_.defaults=createAssigner(_.allKeys,true);

	/*Creates an object that inherits from the given prototype object.If addittional properties are provided then they will be added to the created object.*/
	_.create=function(prototype,props){
		var result=baseCreate(prototype);
		if(props)_.extendOwn(result,props) return result;
	};

	/*Create a(shallow-cloned)duplicate of an object.*/
	_.clone=function(obj){
		if(!_.isObject(obj)) return obj;
		return _.isArray(obj)?obj.slice():_.extend({},obj);
	};

	/*Invokes interceptor with the obj,and then returns obj.The primary of this method is to "tap into" a method chain ,in order to perform operations on intermediate results within the chain.*/
	_.tap=function(obj,interceptor){interceptor(obj);
		return obj;
	};

	/*Returns whether an object has a given set of key:value pairs.*/
	_.isMatch=function(object.attrs){
		var keys=_.keys,length=keys.length;
		if(object==null) return !length;
		var obj=Object(object);
		for(var i=0;i<length;i++){
			var key=keys[i];
			if(attrs[key]!==obj[key]||!(key in obj)) return false;
		}
		return true;
	};

	/*Internal recursive comparison function for isEqual.*/
	var eq=function(a,b,aStack,bStack){
		/*Identical objects are  equal.0===-0,but they aren't identical.See the Harmony egal proposal.*/
		if(a===b) return a!==0||1/a===1/b;
		/*A strict comparison is necessary because null==undefined.*/
		if(a==null||b==null)return a===b;
		/*Unwrap any wrapped objects.*/
		if(a instanceof _) a=a._wrapped;
		if(b instanceof _) b=b._wrapped;

		/*Compare [[Class]] names.*/
		var className=toString.call(a);
		if(className!==toString.call(b)) return false;
		switch(className){

			/*String,numbers,regular expressions,dates,and booleans are compared by value.*/
			case '[object RegExp]':
			/*RegExps are coerced to string for comparison(Note:''+/a/i==='/a/i')*/
			case '[object String]':

			/*Primitives and their corresponding object wrappers are equivalent;thus,"5" is equivalent to new String ("5").*/
			return ''+a===''+b;
			case '[object Number]':

			/*NaN s are equivalent,but non-reflexive.Object(NaN) is equivalent to  NaN*/
			if(+a!==+a) return +b!==+b;

			/*An egal comparison is performed for other numevic valuse.*/
			return +a===0?1/+a===1/b:+a===+b;
			case '[object Date]':
			case '[object Boolean]':

			/*Coerce dates and booleans to numeric primitive values.Dates are compared by their millisecond representations.Note that invalid dates with millisecond representations of NaN are not equivalent.*/
			return +a===+b;
		}

		var areArrays=className==='[object Array]';
		if(!areArrays){
			if(typeof a!='object'||typeof b!='object') return false;

			/*Objects with different constructor are not equivalent,but Object or Array s from different frames are.*/
			var aCtor=a.constructor,bCtor=b.constructor;
			if(aCtor!==bCtor&&!(_.isFunction(aCtor)&&aCtor instanceof aCtor &&_.isFunction(bCtor)&&bCtor instanceof bCtor)&&('constructor' in a &&'constructor' in b)){
				return false;
			}
		}

		/*Initializing stack of traversed objects.It's done here since we only need them of objects and arrays comparison.*/
		aStack=aStack||[];
		bStack=bStack||[];
		var length=aStack.length;
		while(length--){
			/*Linear searck. Performance is inversely proportional to the number of unique nested structures.*/
			if(aStack[length]===a)
				return bStack[length]===b;
		}
		/*Add the first object to the stack of traversed objects.*/
		aStack.push(a);
		bStack.push(b);
		/*Recursively compare objects and arrays.*/
		if(areArrays){
			/*Compare array lengths to determine if a deep comparison is necessay.*/
			length=a.length;
			if(length!==b.length) return false;
			/*Deep compare the contents,Ignoring non-numeric properties.*/
			while(length--){
				if(!eq(a[length],b[length],aStack,bStack)) return false;
			}
		}else{
			/*Deep compare objects.*/
			var keys=_.keys(a),key;
			length=keys.length;

			/*Ensure that both objects contain the same number of properties before comparing deep equality.*/
			if(_.keys(b).length!==length)return false;
			while(length--){
				/*Deep compare each member.*/
				key=keys[length];
				if(!(_.has(b,key)&&eq(a[key],b[key],aStack,bStack))) return false;
			}
		}
		/*Remove the first object from the stack of traversed objects.*/
		aStack.pop();
		bStack.pop();
		return true;
	};

	/*Perform a deep comparison to check if two objects are equal.*/
	_.isEqual=function(a,b){
		return eq(a,b);
	};

	/*Is a given array,string,or object empty? An "empty" object has no enumerable(枚举) own-properties.*/
	_.isEmpty=function(obj){
		if(obj==null) return true;
		if(isArrayLike(obj)&&(_.isArray(obj)||_.isString(obj)||isArguments(obj))) return obj.length===0;
		return _.keys(obj).length===0;
	};

	/*Is a given value a DOM element?*/
	_.isElement=function(obj){
		return !!(obj && obj.nodeType===1);
	};

	/*Is a given value an array? Delegates to ECMA5's native Array.isArray*/
	_.isArray=nativeIsArray||function(obj){
		return toString.call(obj)==='[object Array]';
	};

	/*Is a given variable an object?*/
	_isObject=function(obj){
		var type=typeof obj;
		return type==='function'||type==='object'&&!!obj;
	};

	/*Add some isType method:isArguments,isFunction,isString,isNumber,isDate,isRegExp,isError.*/
	_.each(['Arguments','function','String','Number','Date','RegExp','Error'],function(name){
		_['is'+name]=function(obj){
			return toString.call(obj)==='[object'+name+']';
		};
	});

	/*Define a fallback version of the method in browsers (ahem,IE<9),where there isn't any inspectable "Arguments" type.*/
	if(!_.isArguments(arguments)){
		_.isArguments=function(obj){
			return _.has(has,'callee');
		};
	}
}).call(this);
