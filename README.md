
# SET 8 — ATM Transaction Management

## 1. SQL — `atm.sql`

```
CREATE DATABASE bank;
USE bank;

CREATE TABLE account(
 id INT PRIMARY KEY,
 name VARCHAR(50),
 pin VARCHAR(10),
 balance DOUBLE DEFAULT 0
);

CREATE TABLE transactions(
 id INT AUTO_INCREMENT PRIMARY KEY,
 acc_id INT,
 type VARCHAR(20),
 amount DOUBLE
);

INSERT INTO account VALUES(1,'Arun','1234',5000);
```

## 2. Java — `ATM.java`

This single Java file contains all five classes, JDBC connection, PIN validation, balance enquiry, deposit, withdrawal, PIN change, and transaction retrieval.

```

import java.sql.*;
import java.util.Scanner;

class Account {
    static Connection con;
    static {
        try {
            con = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/bank",
                "root", "password");
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }
    }
}

class BalanceEnquiry extends Account {
    void show(int id, String pin) throws Exception {
        PreparedStatement p = con.prepareStatement(
            "SELECT balance FROM account WHERE id=? AND pin=?");
        p.setInt(1,id);
        p.setString(2,pin);
        ResultSet r=p.executeQuery();
        if(r.next()) System.out.println("Balance: "+r.getDouble(1));
        else System.out.println("Invalid PIN");
    }
}

class Withdraw extends Account {
    void run(int id, String pin, double amount) throws Exception {
        PreparedStatement p=con.prepareStatement(
            "UPDATE account SET balance=balance-? WHERE id=? AND pin=? AND balance>=?");
        p.setDouble(1,amount); p.setInt(2,id);
        p.setString(3,pin); p.setDouble(4,amount);
        if(p.executeUpdate()>0) {
            transaction(id,"Withdraw",amount);
            System.out.println("Withdrawal successful");
        } else System.out.println("Invalid PIN or insufficient balance");
    }
    static void transaction(int id,String type,double amount) throws Exception {
        PreparedStatement p=con.prepareStatement(
            "INSERT INTO transactions(acc_id,type,amount) VALUES(?,?,?)");
        p.setInt(1,id); p.setString(2,type); p.setDouble(3,amount);
        p.executeUpdate();
    }
}

class Deposit extends Account {
    void run(int id,double amount) throws Exception {
        PreparedStatement p=con.prepareStatement(
            "UPDATE account SET balance=balance+? WHERE id=?");
        p.setDouble(1,amount); p.setInt(2,id);
        if(p.executeUpdate()>0) {
            Withdraw.transaction(id,"Deposit",amount);
            System.out.println("Deposit successful");
        }
    }
}

class PinChange extends Account {
    void run(int id,String oldPin,String newPin) throws Exception {
        PreparedStatement p=con.prepareStatement(
            "UPDATE account SET pin=? WHERE id=? AND pin=?");
        p.setString(1,newPin); p.setInt(2,id); p.setString(3,oldPin);
        System.out.println(p.executeUpdate()>0 ?
            "PIN changed" : "Invalid old PIN");
    }
}

public class ATM {
    public static void main(String[] args) throws Exception {
        Scanner s=new Scanner(System.in);
        System.out.print("Account ID: ");
        int id=s.nextInt();
        System.out.print("PIN: ");
        String pin=s.next();
        System.out.println("1.Balance 2.Withdraw 3.Deposit 4.Change PIN 5.Transactions");
        int ch=s.nextInt();

        switch(ch) {
            case 1: new BalanceEnquiry().show(id,pin); break;
            case 2:
                System.out.print("Amount: ");
                new Withdraw().run(id,pin,s.nextDouble()); break;
            case 3:
                System.out.print("Amount: ");
                new Deposit().run(id,s.nextDouble()); break;
            case 4:
                System.out.print("New PIN: ");
                new PinChange().run(id,pin,s.next()); break;
            case 5:
                PreparedStatement p=Account.con.prepareStatement(
                    "SELECT type,amount FROM transactions WHERE acc_id=?");
                p.setInt(1,id);
                ResultSet r=p.executeQuery();
                while(r.next())
                    System.out.println(r.getString(1)+" "+r.getDouble(2));
        }
    }
}

```

Run it: Install MySQL Connector/J, change `password` to your MySQL password, then compile and run:

```
javac -cp ".;mysql-connector-j.jar" ATM.java
java -cp ".;mysql-connector-j.jar" ATM
```

On macOS/Linux, use `:` instead of `;` in the classpath.

## 3. Standalone HTML — `atm.html`

This is a simple frontend demonstration. It runs independently in a browser; it does not connect directly to the Java program or MySQL.

```

<!DOCTYPE html>
<html>
<head>
<title>ATM</title>
<style>
body{font-family:Arial;background:#eef2f7;text-align:center;padding:40px}
main{background:white;padding:25px;margin:auto;max-width:320px;border-radius:12px}
input,button{padding:10px;margin:7px;width:85%}
button{background:#1769aa;color:white;border:0;cursor:pointer}
</style>
</head>
<body>
<main>
<h2>ATM Banking</h2>
<input id="pin" type="password" placeholder="Enter PIN">
<input id="amount" type="number" placeholder="Amount">
<button onclick="act('Balance')">Balance Enquiry</button>
<button onclick="act('Deposit')">Deposit</button>
<button onclick="act('Withdraw')">Withdraw</button>
<button onclick="act('Pin Change')">Change PIN</button>
<p id="msg">Welcome!</p>
</main>
<script>
function act(action){
 let pin=document.getElementById('pin').value;
 let amount=Number(document.getElementById('amount').value);
 if(pin!=="1234"){msg.textContent="Invalid PIN";return;}
 if(action==="Withdraw"&&amount<=0){msg.textContent="Enter valid amount";return;}
 if(action==="Deposit"&&amount<=0){msg.textContent="Enter valid amount";return;}
 msg.textContent=action+" selected. Connect Java backend to process.";
}
</script>
</body>
</html>

```

Lab note: The Java code performs the actual database operations. The HTML is only a UI sample; connecting it requires a web backend.

# SET 9 — Customer Service Portal using Spring Boot

## 1. SQL — `customer.sql`

```
CREATE DATABASE customerdb;
```

Spring Boot will create the table automatically using Hibernate.

## 2. Maven — `pom.xml`

Create a Spring Boot project with Spring Web, Spring Data JPA, Validation, and MySQL Driver dependencies. Add this database configuration to `src/main/resources/application.properties`:

```
spring.datasource.url=jdbc:mysql://localhost:3306/customerdb
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=update
```

## 3. Single Java file — `CustomerApp.java`

Place this file in your Spring Boot project's main Java package. It includes the customer model, repository, and CRUD REST endpoints.

```

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.jpa.repository.*;
import org.springframework.stereotype.*;
import jakarta.persistence.*;
import java.util.*;

@SpringBootApplication
public class CustomerApp {
    public static void main(String[] args) {
        SpringApplication.run(CustomerApp.class,args);
    }
}

@Entity
class Customer {
    @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
    public Long id;
    public String name;
    public String email;
    public String phone;
}

interface CustomerRepo extends JpaRepository<Customer,Long> {}

@RestController
@RequestMapping("/customers")
class CustomerController {
    private final CustomerRepo repo;

    CustomerController(CustomerRepo repo) {
        this.repo=repo;
    }

    @PostMapping
    public Customer add(@RequestBody Customer c) {
        return repo.save(c);
    }

    @GetMapping
    public List<Customer> view() {
        return repo.findAll();
    }

    @PutMapping("/{id}")
    public Customer update(@PathVariable Long id,
                           @RequestBody Customer c) {
        Customer old=repo.findById(id).orElseThrow();
        old.name=c.name;
        old.email=c.email;
        old.phone=c.phone;
        return repo.save(old);
    }

    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {
        if(!repo.existsById(id)) return "Customer not found";
        repo.deleteById(id);
        return "Customer deleted successfully";
    }
}

```

## 4. Standalone HTML — `customer.html`

This page demonstrates adding, viewing, updating, and deleting customers through the Spring Boot API.

```

<!DOCTYPE html>
<html>
<head><title>Customer Portal</title></head>
<body>
<h2>Customer Service Portal</h2>
<input id="id" placeholder="Customer ID">
<input id="name" placeholder="Name">
<input id="email" placeholder="Email">
<input id="phone" placeholder="Phone">
<button onclick="save('POST')">Add</button>
<button onclick="save('PUT')">Update</button>
<button onclick="removeCustomer()">Delete</button>
<button onclick="view()">View All</button>
<pre id="out"></pre>
<script>
const api="http://localhost:8080/customers";
async function save(method){
 let id=document.getElementById("id").value;
 let c={
  name:document.getElementById("name").value,
  email:document.getElementById("email").value,
  phone:document.getElementById("phone").value
 };
 let url=method==="PUT"?api+"/"+id:api;
 let r=await fetch(url,{
  method,headers:{"Content-Type":"application/json"},
  body:JSON.stringify(c)
 });
 document.getElementById("out").textContent=
  r.ok?JSON.stringify(await r.json()):"Operation failed";
}
async function view(){
 let r=await fetch(api);
 document.getElementById("out").textContent=
  r.ok?JSON.stringify(await r.json(),null,2):"Error";
}
async function removeCustomer(){
 let r=await fetch(api+"/"+document.getElementById("id").value,
 {method:"DELETE"});
 document.getElementById("out").textContent=
  r.ok?await r.text():"Delete failed";
}
</script>
</body>
</html>

```

Run: Start MySQL, run `CustomerApp` from your Spring Boot project, then open the HTML page through a local web server. If the browser blocks cross-origin requests, configure CORS in Spring Boot.

# SET 10 — Integrated Bank Web Application

Vaagii, this one is similar to SET 8, but uses a web interface with Spring Boot and Hibernate. To keep the lab code short, we'll put the customer, account, and transaction logic into one Java file.

## 1. SQL — `bank.sql`

```
CREATE DATABASE integratedbank;
```

## 2. Java — `BankApp.java`

Use Spring Boot dependencies: Spring Web, Spring Data JPA, and MySQL Driver. Add the same database properties as SET 9, changing the database name to `integratedbank`.

```

import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;
import org.springframework.data.jpa.repository.*;
import jakarta.persistence.*;
import java.util.*;

@SpringBootApplication
public class BankApp {
    public static void main(String[] args) {
        SpringApplication.run(BankApp.class,args);
    }
}

@Entity
class BankAccount {
    @Id public Long id;
    public String name;
    public String pin;
    public double balance;
}

@Entity
class BankTransaction {
    @Id @GeneratedValue(strategy=GenerationType.IDENTITY)
    public Long id;
    public Long accountId;
    public String type;
    public double amount;
}

interface AccountRepo extends JpaRepository<BankAccount,Long> {}
interface TransactionRepo extends JpaRepository<BankTransaction,Long> {}

@RestController
@RequestMapping("/bank")
class BankController {
    private final AccountRepo accounts;
    private final TransactionRepo transactions;

    BankController(AccountRepo a, TransactionRepo t) {
        accounts=a;
        transactions=t;
    }

    @PostMapping("/add")
    public BankAccount add(@RequestBody BankAccount a) {
        return accounts.save(a);
    }

    @DeleteMapping("/{id}")
    public String delete(@PathVariable Long id) {
        if(!accounts.existsById(id)) return "Account not found";
        accounts.deleteById(id);
        return "Account deleted";
    }

    @GetMapping("/{id}")
    public Object balance(@PathVariable Long id,
                          @RequestParam String pin) {
        BankAccount a=accounts.findById(id).orElse(null);
        if(a==null || !a.pin.equals(pin)) return "Invalid account/PIN";
        return a.balance;
    }

    @PostMapping("/transaction/{id}")
    public String transact(@PathVariable Long id,
                           @RequestParam String pin,
                           @RequestParam String type,
                           @RequestParam double amount) {
        BankAccount a=accounts.findById(id).orElse(null);
        if(a==null || !a.pin.equals(pin)) return "Invalid PIN";
        if(amount<=0) return "Invalid amount";

        if(type.equals("Withdraw") && a.balance<amount)
            return "Insufficient balance";

        if(!type.equals("Deposit") && !type.equals("Withdraw"))
            return "Invalid transaction type";

        a.balance += type.equals("Deposit") ? amount : -amount;
        accounts.save(a);

        BankTransaction t=new BankTransaction();
        t.accountId=id; t.type=type; t.amount=amount;
        transactions.save(t);
        return type+" successful. Balance: "+a.balance;
    }

    @PostMapping("/pin/{id}")
    public String changePin(@PathVariable Long id,
        @RequestParam String oldPin, @RequestParam String newPin) {
        BankAccount a=accounts.findById(id).orElse(null);
        if(a==null || !a.pin.equals(oldPin)) return "Invalid old PIN";
        a.pin=newPin;
        accounts.save(a);
        return "PIN changed successfully";
    }

    @GetMapping("/transactions/{id}")
    public List<BankTransaction> history(@PathVariable Long id) {
        return transactions.findAll().stream()
            .filter(t->t.accountId.equals(id)).toList();
    }
}

```

## 3. Standalone HTML — `bank.html`

This single page demonstrates login, profile, balance enquiry, deposits, withdrawals, PIN changes, contact us, and logout. It connects to the Java REST API for the main banking operations.

```

<!DOCTYPE html>
<html>
<head>
<title>Bank Application</title>
<style>
body{font-family:Arial;background:#eef2f7;text-align:center}
main{background:white;padding:20px;margin:30px auto;max-width:380px}
input,button{padding:10px;margin:5px;width:85%}
button{background:#1769aa;color:white;border:0}
</style>
</head>
<body>
<main>
<h2>Integrated Bank</h2>
<input id="id" placeholder="Account ID">
<input id="pin" type="password" placeholder="PIN">
<input id="amount" type="number" placeholder="Amount">
<button onclick="balance()">Login / Balance</button>
<button onclick="tx('Deposit')">Deposit</button>
<button onclick="tx('Withdraw')">Withdraw</button>
<input id="newpin" placeholder="New PIN">
<button onclick="changePin()">Change PIN</button>
<button onclick="profile()">Profile</button>
<button onclick="history()">Transactions</button>
<button onclick="contact()">Contact Us</button>
<button onclick="logout()">Logout</button>
<p id="msg">Welcome to our bank</p>
</main>
<script>
const api="http://localhost:8080/bank";
let logged=false;
async function call(url,options={}){
 try{
  let r=await fetch(api+url,options);
  let t=await r.text();
  document.getElementById("msg").textContent=t;
  return t;
 }catch(e){
  document.getElementById("msg").textContent="Server unavailable";
 }
}
async function balance(){
 let t=await call("/"+id.value+"?pin="+encodeURIComponent(pin.value));
 if(t!==undefined && !isNaN(Number(t))) logged=true;
}
function tx(type){
 if(!logged){msg.textContent="Login first";return;}
 call("/transaction/"+id.value+"?pin="+encodeURIComponent(pin.value)+
 "&type="+type+"&amount="+amount.value,{method:"POST"});
}
function changePin(){
 if(!logged){msg.textContent="Login first";return;}
 call("/pin/"+id.value+"?oldPin="+encodeURIComponent(pin.value)+
 "&newPin="+encodeURIComponent(newpin.value),{method:"POST"});
}
function profile(){
 if(logged) msg.textContent="Account ID: "+id.value;
 else msg.textContent="Login first";
}
function history(){
 if(logged) call("/transactions/"+id.value);
 else msg.textContent="Login first";
}
function contact(){msg.textContent="Contact: support@bank.com";}
function logout(){logged=false;pin.value="";msg.textContent="Logged out";}
</script>
</body>
</html>

```

## Quick lab viva revision

| Set | Main concept                           | Technology              |
| --- | -------------------------------------- | ----------------------- |
| 8   | ATM operations and transaction records | Java + JDBC             |
| 9   | Customer CRUD                          | Spring Boot + JPA       |
| 10  | Integrated banking operations          | Spring Boot + Hibernate |

