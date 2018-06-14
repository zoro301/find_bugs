## Find bugs
### 1.Public enum method unconditionally sets its field  
示例：

	public enum RoleGrade {
     	ROLE_GRADE_GROUP(101, "Test"),
        ROLE_GRADE_REGION(102, "区域");

        private Integer code;
        private String name;

        RoleGrade(Integer code, String name) {
            this.code = code;
            this.name = name;
        }

        public Integer getCode() {
            return code;
        }

        public void setCode(Integer code) {
            this.code = code;
        }
    }

问题：
setCode方法是public且未被使用  
解决：  
This public method declared in public enum unconditionally sets enum field, thus this field can be changed by malicious code or by accident from another package. Though mutable enum fields may be used for lazy initialization, it's a bad practice to expose them to the outer world. Consider removing this method or declaring it package-private.

---
### 2.Enum field is public and mutable(可变)
示例：  

	public enum RiskSection {
        Direct_Delivery(1, "Test"),

        public Integer code;
        public String name;

        RiskSection(Integer code, String name) {
            this.code = code;
            this.name = name;
        }

        public Integer getCode() {
            return code;
        }

        public void setCode(Integer code) {
            this.code = code;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
问题：  
code可变且是public  
解决：  
A mutable public field is defined inside a public enum, thus can be changed by malicious code or by accident from another package. Though mutable enum fields may be used for lazy initialization, it's a bad practice to expose them to the outer world. Consider declaring this field final and/or package-private.

---
### 3.Non-transient non-serializable instance field in serializable class
示例：

	public class Test implements Serializable {

    	private String id;

    	private List<TestDto> testList;

    	public String getId() {
        	return id;
    	}

	    public void setId(String id) {
	        this.id = id;
	    }

	    public String getName() {
	        return name;
	    }

	    public void setName(String name) {
	        this.name = name;
	    }

	    public List<TestDto> getTestList() {
	        return testList;
	    }

	    public void setTestList(List<TestDto> testList) {
	        this.testList = testList;
	    }
	}
问题：  
testList不可序列化  
解决：  
This Serializable class defines a non-primitive instance field which is neither transient, Serializable, or java.lang.Object, and does not appear to implement the Externalizable interface or the readObject() and writeObject() methods.  Objects of this class will not be deserialized correctly if a non-Serializable object is stored in this field.  

---
### 4.Equals method should not assume anything about the type of its argument  
示例：  

	public class BlameRouteDto implements Serializable {

		private static final long serialVersionUID = -6426119633617606228L;

		private Integer deptId;		
	    private int siteType;

		public Integer getDeptId() {
			return deptId;
		}
		public void setDeptId(Integer deptId) {
			this.deptId = deptId;
		}

	    public int getSiteType() {
	        return siteType;
	    }
	    public void setSiteType(int siteType) {
	        this.siteType = siteType;
	    }

	    public boolean equals(Object obj) {
			BlameRouteDto dto = (BlameRouteDto) obj;

			if (this.siteType != dto.getSiteType()) {
				return false;
			}

			return true;
		}

	    public int hashCode(){
	        return this.deptId;
	    }
	}
问题：  
equals方法中 BlameRouteDto dto = (BlameRouteDto) obj;  
解决：  
The equals(Object o) method shouldn't make any assumptions about the type of o. It should simply return false if o is not the same type as this.  
应该判断一下 obj是不是属于这个类型

---
### 5.Static initializer creates instance before all static final fields assigned
示例：  

	public class Test {
	    public static void main(String[] args)
	    {
	        staticFunction();
	    }

	    static Test st = new Test();

	    static
	    {
	        System.out.println("8");
	    }

	    {
	        System.out.println("1");
	    }

	    {
	        System.out.println("2");
	    }

	    {
	        System.out.println("6");
	    }

	    Test()
	    {
	        System.out.println("3");
	        System.out.println("a="+a+",b="+b);
	    }

	    public static void staticFunction(){  System.out.println("4");
	    }

	    int a=110;
	    static int b =112;
	}
问题：   
static Test st = new Test(); 在在所有的static final字段赋值之前去使用静态初始化的方法创建一个类的实例。  
PS：实例初始化顺序    
1. 基类静态代码块，基类静态成员字段 （并列优先级，按代码中出现先后顺序执行）（只有第一次加载类时执行）  
2. 派生类静态代码块，派生类静态成员字段 （并列优先级，按代码中出现先后顺序执行）（只有第一次加载类时执行）  
3. 基类普通代码块，基类普通成员字段 （并列优先级，按代码中出现先后顺序执行  ）
4. 基类构造函数  
5. 派生类普通代码块，派生类普通成员字段 （并列优先级，按代码中出现先后顺序执行）  
6. 派生类构造函数

---
### 6.May expose internal representation by returning reference to mutable object  
示例：

	public class CompletedRateQueryParam extends GoldShieldAppBaseParam implements Serializable {

	    private Date date;  

	    public Date getDate() {
	        return date;
	    }

	    public void setDate(Date date) {
	        this.date = date;
	    }
   	}

问题：  
date是可变类型，getter、setter方法会得到或通过对可变对象的引用操作而暴露代码内部实现  
解决：  
Returning a reference to a mutable object value stored in one of the object's fields exposes the internal representation of the object.  If instances are accessed by untrusted code, and unchecked changes to the mutable object would compromise security or other important properties, you will need to do something different. Returning a new copy of the object is better approach in many situations.

---
### 7.Boxed value is unboxed and then immediately reboxed
示例：  

	   public Integer getStart() {
	        Integer _pageNo = (curPageBlame == null ? 1 : curPageBlame);
	        Integer _pageSize = (pageSizeBlame == null ? 20 : pageSizeBlame);
	        return (_pageNo - 1) * _pageSize;
   		}
问题：  
Integer _pageNo = (curPageBlame == null ? 1 : curPageBlame); 装箱的值被拆箱，赋值后又重新装箱了，性能问题  
解决：  
可以统一类型，Integer _pageNo = (curPageBlame == null ?Integer.valueOf(1) : curPageBlame);

---
### 8.Method invokes inefficient(效率低下的) new String(String) constructor
示例：  

	String s1 = new String(“myString”);
问题：  
效率低  
解决：  
String s1 = "myString";  
new的方式浪费内存,在程序编译期，编译程序先去字符串常量池检查，是否存在“myString”,如果不存在，则在常量池中开辟一个内存空间存放“myString”；如果存在的话，则不用重新开辟空间，保证常量池中只有一个“myString”常量，节省内存空间。然后在内存堆中开辟一块空间存放new出来的String实例，在栈中开辟一块空间，命名为“s1”，存放的值为堆中String实例的内存地址，这个过程就是将引用s1指向new出来的String实例。

---
### 9.Method concatenates strings using + in a loop
示例：

 	for (int i = 0, len = rlCodes.length; i < len; i++) {
        String code = rlCodes[i];
        if (StringUtils.isEmpty(code) || !NumberUtils.isNumber(code)) {
            continue;
        }
        result += getRiskLabelName(Integer.parseInt(code)) + "<br>";
    }
问题：  
循环拼接字符串时使用String类型，String类型不可变，每次拼接都会生产新的对象 。   
解决：  
使用StringBuffer/StringBuilder替换 。   
The method seems to be building a String using concatenation in a loop. In each iteration, the String is converted to a StringBuffer/StringBuilder, appended to, and converted back to a String. This can lead to a cost quadratic in the number of iterations, as the growing string is recopied in each iteration.
Better performance can be obtained by using a StringBuffer (or StringBuilder in Java 1.5) explicitly.  
For example:  

	  // This is bad
	  String s = "";  
	  for (int i = 0; i < field.length; ++i) {  
	    s = s + field[i];  
	  }  

	  // This is better  
	  StringBuffer buf = new StringBuffer();  
	  for (int i = 0; i < field.length; ++i) {  
	    buf.append(field[i]);  
	  }    
	  String s = buf.toString();

---
### 10.Inefficient use of keySet iterator instead of entrySet iterator
示例:

	 for (int risk_point : riskPiontMap.keySet()) {
            Map<String, Integer> org_map = riskPiontMap.get(risk_point);
	 }
问题：  
使用entrySet 替换 keySet 方法，效率更高。  
解决：  
This method accesses the value of a Map entry, using a key that was retrieved from a keySet iterator. It is more efficient to use an iterator on the entrySet of the map, to avoid the Map.get(key) lookup.  

---
### 11.Should be a static inner class
示例：  

	public class ConstantUtil {

	    public static final String USER_RESSTATUS = "resStatus";

	    public class EsConstants {

	        public static final String INFO_INDEX = "test";
		}
	}
问题：  
EsConstants应该是静态内部类  
解决：  
若成员类中未访问外围类的非静态成员，为避免额外的空间和时间开销，建议改用静态成员类。非静态成员类可以访问外围类的任何成员，但前提是必须存在外围类对象。JAVA需要额外维护非静态成员类和外围类对象的关系。     
This class is an inner class, but does not use its embedded reference to the object which created it.  This reference makes the instances of the class larger, and may keep the reference to the creator object alive longer than necessary.  If possible, the class should be made static.

---
