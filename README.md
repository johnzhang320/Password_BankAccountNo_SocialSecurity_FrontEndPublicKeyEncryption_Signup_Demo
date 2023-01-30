## Use Frontend Public Key to encrypt Password, Bank Account # and Social Security
### Introduce an Useful Signup Code Implementation and Demo 
## Overview

### Why and how do we apply frontend encryption ?
   
   Https (TLS) be able to encrypt internet channel to prevent the hackers from stealing sensitive data in network, however, 
   some invaded viruses, like Trojan Horse and Active-X, still are able to sneak into your local machine to steal the data
   by means of harmless links, misleading email,a falsified website, or a fake advertisement.
      
   They are using keylogger to make system call to log the keystroke, recording critical javascript variables or using 
   hugh Hash Strings of 'Rainbow table' to guess your hashed password sent be UI
      
   We attempt to hide the sensitive form field variables and when we complete enter sensitive field and leave focus from 
   the field, encrypt the data that you just entered, dynamically obtain pubic key from server to encrypt form field by 
   javascript. 


### Why do we use Public Key to encrypt sensitive data in frontend?
  
   First of all, some sensitive data such as BankAccount number and Social Security number must be decrypted in server 
   side for further business use. So apply public key cryptography to be able to decrypt those data to plain text in 
   Server side and encrypt the data in frontend before user submit the data form.
   
   Secondly, signup pages normally consist of first page for creating username and password and press 'Continue' to next
   pages , until getting 'Submit' page, sensitive information plain texts will stay for a while between pages, which 
   give the invaded virus a timing chance to steal them, on this mechinism, people leave those fields immediately encrypt
   to eliminate the chance for those viruses to steal the information

   Regarding of the password, some developers may use one time hash algorithm, such as SHA256 or MD5, to hash password 
   in UI and send to server save it to Database.
       
   This way leaves the room for "Rain Table" which holds Hugh Hashed Strings to brute force 'Guess' hashed password, we 
   use public key cryptography to be able to avoid this hash "Guess' because PublicKey-encrypted RSA String is different 
   from those Hashed String and even more complicated(1024, 2048 bytes etc). 
       
   On this mechanism, each time when a user loads the Signup page, the server will generate a new public key which is 
   different from previous public key as users loaded the same Signup page previously.
    
   Since Spring Boot 2.0, Spring Security uses BCryptPasswordEncoder to save salt hashed password, for the same password 
   plain text, this encoder generates a different encoded string, we save this encoded password string to the database. 
   Rain Table is hard to guess the changeable hashed password.
    
   The method: BCryptPasswordEncoder.matchers(passwordPlainText, database_saved_bcrypted_password) required passwordPlainText, 
   FEPKE mechanism encrypts password in the UI side and provide a decrypt method in server side to get password plain 
   text, then for the important password handling--password validation, we can validate wheather same user try to create the 
   same password as he/she already created !
    
   In order to take advantage of BCryptPasswordEncoder, We can not SHA256 ot MD5 hash passwords in UI and use public key 
   to solve this problem.
   

   
## Project Structure   

   ![](images/project_structure.png)  
   
## Running Environment and Development Tools 

   JDK1.8
   
   SpringBoot 2.1.3.RELEASE   
 
   Intellij Community Verson
   
   Postman
   
   Any Browser
   
   #Especially ensure to setup Intellij Project Structure to JDK 1.8 for key generator
   
   ![](images/Intellij_project_sdks_in_jdk1.8.png)

## Dependencies
### Major Dependencies
    
    org.bouncycastle.bcprov-jdk16.1.45  --- Generates Public Key Pair, Private Key Pair, descrypts FEPKEed data to plain text
    
    Spring boot 2.1.3
    Spring boot Web                     --- Spring MVC for Demo 
    org.apache.tile.tiles-jsp.3.0.5     --- Support view header, menu, body and footer
    Spring boot Security                --- Apply BCryptPasswordEncoder for password saving and password validation
    Spring boot JPA Data / MySQL        --- Signup data Model database access
    
    org.modelmapper.2.3.5               --- Model To Dto or Dto to Model conversion
    
    org.projectlombok                   
    
    
   
  ...  
  
       ..................
  
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.3.RELEASE</version>
	</parent>

	<properties>
		<java.version>1.8</java.version>
		<spring-cloud.version>Greenwich.RELEASE</spring-cloud.version>
		<start-class>com.front.end.pk.encrypt.demo.FrontEndCryptionDemoApplication</start-class>
	</properties>
	<dependencies>

		<dependency>
			<groupId>org.bouncycastle</groupId>
			<artifactId>bcprov-jdk16</artifactId>
			<version>1.45</version>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-security</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<dependency>
			<groupId>org.projectlombok</groupId>
			<artifactId>lombok</artifactId>
			<optional>true</optional>
		</dependency>
		<dependency>
			<groupId>org.modelmapper</groupId>
			<artifactId>modelmapper</artifactId>
			<version>2.3.5</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.apache.tiles</groupId>
			<artifactId>tiles-jsp</artifactId>
			<version>3.0.5</version>
		</dependency>


		.......

	</dependencies>
   
   ...

   
### Workflow Chart

  ![](images/FrontEndPublicKeyEncryptionDetail.jpg)
  
  ## Based on the sequence No. (1), (2),(3).....(21) of each arrows in workflow chart, we will explain function with description, the code and demo
  
  
  
  ### (1) - Signup Request 
     
      http://localhost:8080/FrontEndPublicKeyEncryption/signup
      
      FrontEndPublicKeyEncryption is defined as contextPath in application.properties
      
      server.servlet.contextPath=/FrontEndPublicKeyEncryption
      
      If apply Spring Security, run any web page popout its default login page, before Spring Boot 2.7.8, we can use 
      WebSecurityConfigurerAdapter disable default login page and allow /signup and /getKeypair.html to work without 
      authentication 
      
      
      ...
        
     @EnableWebSecurity
     @Configuration
     public class SecurityConfig extends WebSecurityConfigurerAdapter {

        @Override
        protected void configure(HttpSecurity http) throws Exception
        {
            http.csrf().disable()
            .authorizeRequests()
                  .antMatchers(
                          "/",
                          "/signup",
                          "/savePassword",
                        "/getKeyPair.html").permitAll()
                  .and().formLogin().disable();

         }
     }

      
      ...
      
      
      
  
  (2) Spring MVC Controller get such request, load signup page:
   
 ...
 
   	@RequestMapping(value="/signup",method = RequestMethod.GET)
	public ModelAndView signupForm(ModelAndView modelAndView)
			throws Exception {
		modelAndView.setViewName("FrontEndCryptionDemo");
		modelAndView.addObject("agentTableRequestDto",new AgentTableDto());
		/**
		 *  Return to tile definition name: AgentLogin defined in tiles.xml
		 */
		return modelAndView;
	}
	
 ...
 
  
  Here FrontEndCryptionDemo is Signup page handler points signup definition in tiles.xml, signup page body code is FrontEndCryptionDemo.jsp, coming 
  with header.jsp and footer.jsp (see code source)

## (3) Before Load JSP HTML context, send Public Key request
   
   stringCryption.getPublicKey("/FrontEndPublicKeyEncryption/getKeyPair.html") send public key request to keypairManager via Rest API
   
  ...
     <script type="text/javascript" src="js/lib/jquery-1.8.0.js"></script>
     <script type="text/javascript" src="js/lib/jquery.jcryption-1.1.js"></script> 
     
       <!-- Front End Encryption Customized Code -->
     <script type="text/javascript"	src="js/utils/stringCryption.js">		
     </script>
       <script type="text/javascript"> 
	  $(document).ready(function(){
		stringCryption.getPublicKey("/FrontEndPublicKeyEncryption/getKeyPair.html")
	 	stringCryption.initialize("password"); 		  
		stringCryption.initialize("creditNumber"); 		  
		stringCryption.initialize("socialSecurity");
	  }); 
  </script>  
  
  ...
  
### (4) KeyPairManager generates Public Key pair (e,n) and model and Private Key pair

  ...

     package com.front.end.pk.encrypt.demo.fepke_api;

     import lombok.Data;
     import lombok.extern.slf4j.Slf4j;
     import java.security.KeyPair;

     @Slf4j
     @Data  
     public class KeyPairManager {

        private static KeyPairManager handler=null;	
        private KeyPair keyPair=null; 
        private String keyString=null;
        JCryptionUtil jCryptionUtil =null;
	
        public synchronized static KeyPairManager getInstance() {	
		if (null==handler) {
		    handler = new KeyPairManager();
		}
		return handler;
	}
	
	 /**
	  *  KeyPair structure: {e,n} is public key , {d,n} is private key, 
	  *  md=(p-1) x (q-1) , n=p x q, p and q must be prime number 
	  */
	  
			
        private KeyPairManager() {
		try {
			 jCryptionUtil = new JCryptionUtil();  	       
			 keyPair = jCryptionUtil.generateKeypair();  
			 StringBuffer output = new StringBuffer();  
			 String e = jCryptionUtil.getPublicKeyExponent(keyPair);  
			 String n = jCryptionUtil.getPublicKeyModulus(keyPair);  
			 String md = String.valueOf(jCryptionUtil.getMaxDigits());  		
			 output.append("{\"e\":\"");  
			 output.append(e);  
			 output.append("\",\"n\":\"");  
			 output.append(n);  
			 output.append("\",\"maxdigits\":\"");  
			 output.append(md);  
			 output.append("\"}");  
			 output.toString();  
			 keyString = output.toString().replaceAll("\r", "").replaceAll("\n", "").trim();  
		} catch (Exception e) {
			log.info("Generate Key Failed because of "+e.getMessage());
		}
	}

	public String decrypt(String encrypted) {
		String retVal = null;
		log.debug("encrypted String="+encrypted+"\n keyPair="+keyPair);
		try {
			retVal = jCryptionUtil.decrypt(encrypted, keyPair);
		} catch (Exception e) {
			log.info("Descryption Failed because of "+e.getMessage());
		}
		return retVal;
	 }
    }

  
  ...
  
  
   
  ### (5) Here is an sample to explain Public Key RSA Cryptography 
  
  ![](images/RSA_Cryptography.png)

  In above diagram, (e,n) is public key pair and (d,n) is private key pair, M is plainText, public key encrypt is C=(M^e) mod n , 
  private key decrypt is D = (C^d) mod n.  
  
  /FrontEndPublicKeyEncryption/getKeyPair.html will return (e,n) in asscii character format, this URL is synchronized request
     
### (6) Register Password, Bank A/c and Social Security as Frontend Public Key Encrypting (FEPKE) fields
      
  	 	stringCryption.initialize("password"); 		  
		stringCryption.initialize("creditNumber"); 		  
		stringCryption.initialize("socialSecurity");
		
    I made stringCryption.js as an interface between JSP and Javascript public key encryption library: jquery.jcryption-1.1.js,
    I also made some synchronize ajax call in this library.
    
    
### (7) Spring MVC return the signup page to user as followig empty page which let user enter data

   ![](images/signup_empty_page_for_dto.png)
   
    We call loan agent sigup page, therefore we create agentTableDto to accept user entered data and cipherText data encrypt by 
    Javascript
    
    As we known, Spring boot presentative layer we use DTO to accept raw entered data and supper basic validation work by javax
    validation annotations
    
    ...
    
    package com.front.end.pk.encrypt.demo.dto;

import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Data;
import lombok.NoArgsConstructor;
import org.springframework.format.annotation.DateTimeFormat;
import javax.validation.constraints.*;


/**
 * Agents entity. @author MyEclipse Persistence Tools
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class AgentTableDto {

	@NotBlank(message = "Username is required")
	private String userName;
	
	@NotBlank(message="password is required")
	private String password;

	@Email(message = "Email is invalid", regexp = "[a-z0-9._%+-]+@[a-z0-9.-]+\\.[a-z]{2,3}")
	private String emailAddress;
	
	@NotBlank(message="CreditNumber is required")
	private String creditNumber;
	
	@NotBlank(message = "Credit Card Holder is required")
	private String cardHolderName;
	
	@NotBlank(message = "Expired Date is required")
	@DateTimeFormat(pattern = "mm/yy")
	private String expiringDate;
	
	@NotBlank(message = "Security Code is required")
	private String securityCode;
	
	@NotBlank(message = "Social Security Number is required")
	private String socialSecurity;
	
	@NotBlank(message = "SSO full name is required")
	
	private String fullName;
	
	private Boolean passwordMatched;
	
	private String message;
}
    ...
    

## Getting Started

### Dependencies

* Describe any prerequisites, libraries, OS version, etc., needed before installing program.
* ex. Windows 10

### Installing

* How/where to download your program
* Any modifications needed to be made to files/folders

### Executing program

* How to run the program
* Step-by-step bullets
```
code blocks for commands
```
## Source code download
   https://github.com/johnzhang320/front-end-public-key-encryption
## Help

Any advise for common problems or issues.
```
command to run if program contains helper info
```

## Authors

Contributors names and contact info

ex. Dominique Pizzie  
ex. [@DomPizzie](https://twitter.com/dompizzie)

## Version History

* 0.2
    * Various bug fixes and optimizations
    * See [commit change]() or See [release history]()
* 0.1
    * Initial Release

## License

This project is licensed under the [NAME HERE] License - see the LICENSE.md file for details

## Acknowledgments

       
        



