# SpringBoot2.0�ļ��ϴ�

## 1.�������

```
<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jdbc</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-thymeleaf</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-devtools</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.mariadb.jdbc</groupId>
			<artifactId>mariadb-java-client</artifactId>
		</dependency>
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.35</version>
		</dependency>
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-lang3</artifactId>
			<version>3.6</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
```

## 2.SpringBoot����
```
server.port=8088

# Thymeleaf Setting
spring.thymeleaf.cache=false
spring.thymeleaf.mode=HTML

# ===============
# = DataBase Setting
# ===============

# Connection url for the database "iproduct"
spring.datasource.url = jdbc:mariadb://127.0.0.1:3306/iproduct?useUnicode=true&amp;characterEncoding=utf-8&autoReconnect=true&zeroDateTimeBehavior=round

# DataBase Username and password
spring.datasource.username = root
spring.datasource.password = 123456

# Keep the connection alive if idle for a long time (needed in production)
spring.datasource.testWhileIdle = true
spring.datasource.validationQuery = SELECT 1

# =============
# = JPA/Hibernate Setting
# =============

# Show or not log for each sql query
spring.jpa.show-sql = true

# Hibernate ddl auto (create, create-drop, update): with "update" the database
# schema will be automatically updated accordingly to java entities found in
# the project
spring.jpa.hibernate.ddl-auto = update

# Naming strategy
spring.jpa.hibernate.naming-strategy = org.hibernate.cfg.ImprovedNamingStrategy

# Allows Hibernate to generate SQL optimized for a particular DBMS
spring.jpa.properties.hibernate.dialect = org.hibernate.dialect.MySQL5Dialect

# Spring 2.0.0 file upload setting
spring.servlet.multipart.max-file-size=-1

spring.http.multipart.maxRequestSize=100Mb
```

## 3.ʵ��ͼƬ�ϴ�
### (1)ʵ��
```
@Entity
@Table(name = "product")
public class Product {
	
    @Id
	private String id;
	private String name;      //��Ʒ��
	private String produce;   //��Ʒ����
	private String photo;     //��Ʒ��Ƭ

	public void init(){
		this.id = UUID.randomUUID().toString().replaceAll("-","");
	}
	
	//getters and setters...
	}
```

### ��2��������
 ������Ҫ3������һ������ʾͼƬ�ϴ�����ҳ�ķ�����һ����ͼƬ�ϴ����ܵķ���������һ������ʾ�ϴ�ͼƬ�ķ���
```
 @Autowired
	private ProductService productService;
	
	@GetMapping("/addpro")
	public String addproduct(){
		return "add_product";
	}
	
	@PostMapping("/addpro")
	public String addProduct(ProductForm productForm){
		productService.Addproduct(productForm);
		return "redirect:/product";
	}
	
	@GetMapping("/product")
	public String product(Model m){
	    List list = productService.findAll();
	    m.addAttribute("product", list);
		return "product";
	}
```
 ���ProductForm��������Ҫ���ύ�Ĳ�������װ��������
```
public class ProductForm {

	private String name;  //��Ʒ��
	private String produce;  //��Ʒ����
	private MultipartFile file; //�ϴ����ļ�
	
	//getters and setters...
	}
```

### (3) ҵ���ʵ��
```
@Service
@Transactional
public class ProductService {

	@Autowired
	private ProductDao productDao;
	
	public void Addproduct(ProductForm productForm){
		Product product = new Product();
		product.init();
		BeanUtils.copyProperties(productForm, product, Product.class);
		product.setPhoto(FileUpload.fileUpload(productForm.getFile()));
		productDao.save(product);
	}
	
	public List<Product> findAll(){
		return productDao.findAll();
	}
	
}
```
 ͼƬ�ϴ��ķ������ҷ�װ��һ��FileUpload���ˣ�
```
public class FileUpload {

	public static String fileUpload(MultipartFile File){
		
		        //�ļ�Դ�ļ���
				String fileName = new Date().getTime()+File.getOriginalFilename();
				System.out.println("Դ�ļ���------>"+fileName);
				//�ļ��ϴ�·��
				String filePath = "E:/images/";
				System.out.println("�ļ��ϴ�·��------->"+ filePath);
				File file = new File(filePath);
				if(!file.exists()){
				 file.mkdirs();	
				}
				
				try {
					//���ϴ��ļ�����ͨ��I/O��д�뵽���ļ�
					FileOutputStream out = new FileOutputStream(filePath+fileName);
					out.write(File.getBytes());
					out.flush();
					out.close();
				} catch (FileNotFoundException e) {
					
					e.printStackTrace();
				} catch (IOException e) {
				
					e.printStackTrace();
				}
				
				
				return "http://localhost:8088/images/"+fileName;
	}
	
}
``` 

### (4)����·�����������
����·��
```
@Configuration
public class PhotoConfig implements WebMvcConfigurer {

	 public void addResourceHandlers(ResourceHandlerRegistry registry) {   
		  /**
		 * @Description: ���ļ���·����������,����һ������·��/Path/** ����ֻҪ��<img src="/Path/picName.jpg" />�����ֱ������ͼƬ
		 *����ͼƬ������·��  "file:/+����ͼƬ�ĵ�ַ"
		 * @Date�� Create in 14:08 2017/12/20
		 */     registry.addResourceHandler("/images/**").addResourceLocations("file:/E:/images/");
		       
		    }
}

��������
```
//�����������
@Configuration  
public class CrosConfig implements WebMvcConfigurer{  

    public void addCorsMappings(CorsRegistry registry) {  
        registry.addMapping("/**")  
                .allowedOrigins("*")  
                .allowCredentials(true)  
                .allowedMethods("GET", "POST", "DELETE", "PUT")  
                .maxAge(3600);  
    }

}
```

## 5.��ҳ
��1���ļ��ϴ���ҳ
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<form enctype="multipart/form-data" method="post" th:action="@{/addpro}">
<p>
��Ʒ����   <input type="text" name="name"/>
</p>
��Ʒ���ܣ�
<p> 
   <textarea rows="15" cols="70" name="produce" id="textarea"></textarea>
</p>
<p>
 ͼƬ:<input type="file" name="file"/>
 </p>
    <input type="submit" value="�ϴ�"/>
    </form>
</body>
</html>
```
(2)��ʾ�ļ���ҳ
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
<div th:each="pro:${product}">
<p>
<h1 th:text="${pro?.name}"></h1>
</p>
��Ʒ����:<p th:text="${pro?.produce}"></p>
<img th:src="@{${pro?.photo}}" style="width:500px;height:450px;" />
</div>
</body>
</html>
```



