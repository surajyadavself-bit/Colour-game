npm install express mysql jsonwebtoken bcrypt cors
const jwt = require("jsonwebtoken");
const bcrypt = require("bcrypt");

app.post("/api/register", async (req,res)=>{
  let {username, password} = req.body;
  let hash = await bcrypt.hash(password,10);

  db.query("INSERT INTO users(username,password) VALUES(?,?)",
  [username,hash]);

  res.send("User Created");
});

app.post("/api/login",(req,res)=>{
  let {username,password} = req.body;

  db.query("SELECT * FROM users WHERE username=?",[username],
  async (err,result)=>{
    if(result.length===0) return res.send("User not found");

    let valid = await bcrypt.compare(password,result[0].password);

    if(!valid) return res.send("Wrong password");

    let token = jwt.sign({id:result[0].id},"SECRET");
    res.send({token});
  });
});
app.get("/api/wallet", auth, (req,res)=>{
  db.query("SELECT balance FROM users WHERE id=?",
  [req.user.id], (err,result)=>{
    res.send(result[0]);
  });
});
let bets = [];

app.post("/api/game/bet", auth, (req,res)=>{
  bets.push({
    user:req.user.id,
    color:req.body.color,
    amount:req.body.amount
  });
  res.send("Bet placed");
});

setInterval(()=>{
  let colors=["RED","GREEN","VIOLET"];
  let result = colors[Math.floor(Math.random()*3)];

  bets.forEach(bet=>{
    if(bet.color===result){
      db.query("UPDATE users SET balance=balance+? WHERE id=?",
      [bet.amount*2, bet.user]);
    }
  });

  bets=[];
},30000);
app.get("/api/admin/users",(req,res)=>{
  db.query("SELECT id,username,balance FROM users",(err,result)=>{
    res.send(result);
  });
});
CREATE TABLE withdraws (
 id INT AUTO_INCREMENT PRIMARY KEY,
 user_id INT,
 amount INT,
 status VARCHAR(20) DEFAULT 'pending'
);
app.post("/api/admin/approve",(req,res)=>{
  let {id,user_id,amount} = req.body;

  db.query("UPDATE withdraws SET status='approved' WHERE id=?",[id]);

  db.query("UPDATE users SET balance=balance-? WHERE id=?",
  [amount,user_id]);

  res.send("Approved");
});
<h2>Admin Panel</h2>

<button onclick="loadUsers()">Load Users</button>
<div id="users"></div>

<script>
function loadUsers(){
 fetch("/api/admin/users")
 .then(res=>res.json())
 .then(data=>{
   let html="";
   data.forEach(u=>{
     html+=`<p>${u.username} - â‚¹${u.balance}</p>`;
   });
   document.getElementById("users").innerHTML=html;
 });
}
</script>
