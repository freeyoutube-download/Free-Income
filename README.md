<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Gmail Chat - WhatsApp Style</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Poppins', sans-serif; }
  body { background: #e5ddd5; display: flex; justify-content: center; align-items: center; height: 100vh; }

  .login-container, .app-container { width: 1000px; height: 600px; border-radius: 10px; overflow: hidden; display: flex; }

  /* Login */
  .login-container { flex-direction: column; justify-content: center; align-items: center; background: #075e54; color: white; }
  .login-container input { padding: 10px; margin: 10px 0; border-radius: 5px; border: none; width: 250px; }
  .login-container button { padding: 10px 20px; border: none; border-radius: 5px; background: #25d366; color: white; cursor: pointer; }

  /* App */
  .app-container { display: none; background: white; box-shadow: 0 0 10px rgba(0,0,0,0.2); }
  .sidebar { width: 30%; background: #ededed; display: flex; flex-direction: column; position: relative; }
  .sidebar-header { padding: 20px; background: #075e54; color: white; font-size: 1.2em; display: flex; justify-content: space-between; align-items: center; }
  .three-dots { cursor: pointer; font-size: 1.5em; padding: 0 5px; }
  .dropdown { display: none; position: absolute; top: 60px; left: 5px; background: white; border: 1px solid #ccc; border-radius: 5px; overflow: hidden; z-index: 1000; }
  .dropdown button { display: block; width: 100%; padding: 10px; border: none; background: white; text-align: left; cursor: pointer; }
  .dropdown button:hover { background: #ddd; }
  .contacts { flex: 1; overflow-y: auto; }
  .contact { padding: 15px; border-bottom: 1px solid #ccc; cursor: pointer; }
  .contact:hover { background: #ddd; }

  .main { flex: 1; display: flex; flex-direction: column; }
  .main-header { padding: 15px; background: #075e54; color: white; font-size: 1.2em; }
  .chat-area { flex: 1; padding: 15px; background: #ece5dd; overflow-y: scroll; display: flex; flex-direction: column-reverse; scroll-behavior: smooth; }
  .chat-message { max-width: 60%; margin: 5px 0; padding: 10px; border-radius: 7px; }
  .sent { background: #dcf8c6; align-self: flex-end; }
  .received { background: white; align-self: flex-start; }

  .input-area { display: flex; padding: 10px; background: #f0f0f0; }
  .input-area input { flex: 1; padding: 10px; border-radius: 20px; border: 1px solid #ccc; }
  .input-area button { margin-left: 10px; padding: 10px 20px; border: none; border-radius: 20px; background: #25d366; color: white; cursor: pointer; }
</style>
</head>
<body>

<div class="login-container" id="loginContainer">
  <h2>Sign in with Gmail</h2>
  <input type="email" id="emailInput" placeholder="Enter Gmail">
  <button onclick="signIn()">Sign In</button>
</div>

<div class="app-container" id="appContainer">
  <div class="sidebar">
    <div class="sidebar-header">
      Contacts
      <span class="three-dots" onclick="toggleDropdown()">⋮</span>
    </div>
    <div class="dropdown" id="dropdownMenu">
      <button onclick="addConnect()">Add Connect</button>
      <button onclick="showRequests()">Requests</button>
      <button onclick="customerCare()">Customer Care</button>
    </div>
    <div class="contacts" id="contactsList"></div>
  </div>
  <div class="main">
    <div class="main-header" id="chatHeader">Select a contact</div>
    <div class="chat-area" id="chatArea"></div>
    <div class="input-area">
      <input type="text" id="chatInput" placeholder="Type a message">
      <button onclick="sendMessage()">Send</button>
    </div>
  </div>
</div>

<script>
let loggedInUser = null;
const users = {};
const connections = {};
const requests = {};
let currentChat = null;

function signIn(){
  const email = document.getElementById('emailInput').value.trim();
  if(!email) return alert('Enter Gmail');
  loggedInUser = email;
  document.getElementById('loginContainer').style.display = 'none';
  document.getElementById('appContainer').style.display = 'flex';
  if(!users[email]) users[email] = [];
  if(!connections[email]) connections[email] = [];
  if(!requests[email]) requests[email] = [];
  renderContacts();
}

function toggleDropdown(){
  const menu = document.getElementById('dropdownMenu');
  menu.style.display = menu.style.display === 'block' ? 'none' : 'block';
}

function renderContacts(){
  const list = document.getElementById('contactsList');
  list.innerHTML = '';
  const conns = connections[loggedInUser];
  conns.forEach(c => {
    const div = document.createElement('div');
    div.className = 'contact';
    div.textContent = c;
    div.onclick = ()=> openChat(c);
    list.appendChild(div);
  });
}

function openChat(contact){
  currentChat = contact;
  document.getElementById('chatHeader').textContent = contact;
  renderChat();
}

function renderChat(){
  const chatArea = document.getElementById('chatArea');
  chatArea.innerHTML = '';
  const msgs = users[currentChat] || [];
  msgs.slice().reverse().forEach(m => {
    const div = document.createElement('div');
    div.className = 'chat-message ' + (m.sentBy===loggedInUser?'sent':'received');
    div.textContent = m.text;
    chatArea.appendChild(div);
  });
  chatArea.scrollTop = chatArea.scrollHeight;
}

function sendMessage(){
  const input = document.getElementById('chatInput');
  const text = input.value.trim();
  if(!text || !currentChat) return;
  if(!users[currentChat]) users[currentChat] = [];
  users[currentChat].push({text, sentBy: loggedInUser});
  renderChat();
  input.value = '';
}

function addConnect(){
  const email = prompt('Enter Gmail to connect');
  if(!email || email===loggedInUser) return;
  if(!users[email]) return alert('User not signed in');
  if(!requests[email].includes(loggedInUser)){
    requests[email].push(loggedInUser);
    alert('Connect request sent to '+email);
  } else alert('Request already sent');
}

function showRequests(){
  const reqs = requests[loggedInUser];
  if(!reqs.length) return alert('No requests');
  reqs.forEach(r=>{
    const accept = confirm('Accept request from '+r+'?');
    if(accept){
      if(!connections[loggedInUser].includes(r)) connections[loggedInUser].push(r);
      if(!connections[r].includes(loggedInUser)) connections[r].push(loggedInUser);
      alert('Connected with '+r);
    } else alert('Rejected '+r);
  });
  requests[loggedInUser]=[];
  renderContacts();
}

function customerCare(){
  alert('Write your problem to this Gmail: mdabir12re@gmail.com');
}
</script>

</body>
</html>
</html>
