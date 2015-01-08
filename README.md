Framelocker API Docs
========

<h3>Navigation</h3>

1. [Server API](#server-api)
2. [NodeJS Chat API](#chat-api-docsnodejs-socket)
3. [NodeJS Chat API Invitation](#invitation-for-chat)
4. [WOD Chat specialities](#wod-chat-specialties)
5. [Chat statistic](#chat-statistics)

#Server API

<h4>API admin page - http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/novp/</h4>
<h4>API access - http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/api</h4>
<h4>Table 1.1 - API methods</h4>

 # | Method        		  | TYPE | Request                                                        | Response                              |
---|----------------------|------|----------------------------------------------------------------|---------------------------------------|
 1 | upload_file  		  | POST | {method:upload_file, token, novp_file,[title],[description]}   | {status, description, [title, src]}   |
 2 | get_files   		  | GET  | {method:get_files, token}                                      | {status, description,                 |
   |			 		  |	     |  														 	  | [file_data:{id,user_id,bucket_id,     |
   |             		  |      |                                                        		  | filename,size,ext,aws,date}]}         |
 3 | register      		  | POST | {method:"register", params: {username, password, [novp_file]}} | {status, description}                 |     
 4 | signin      		  | POST | {method:"signin", params:{login, pass}}                        | {status, description, <b>token</b>}   |
 5 | signout      		  | POST | {method:"signout", token}                                      | {status, description}                 |         
 6 | upload_avatar 		  | POST | {method:"upload_avatar", token, novp_file}                     | {status, description, [filename]}     |          
 7 | set_name      		  | POST | {method:"set_name", token, params: {fstname, lstname}}         | {status, description}                 |      
 8 | add_user             | POST | {method:"add_user", token, params: {name,role=7,email,message}}| {status, description}                 |      
 9 | assigning_to_facebook| POST | {method:"assigning_to_facebook", token, fid}                   | {status, description}                 |
 10| prime_check          | GET  | {method:"prime_check", token}                                  | {status, description}                 |
   |  		              |      |    <h3>For WOD chat</h3>                                       |                                       | 
 11| get_boxes		      | GET  | {method:"get_boxes", token}                                    | {status, description, boxes}          |
 12| assign_box    		  | POST | {method:"assign_box", token, params: {uid, box}}               | {status, description}                 |
 
 <h4>Method details</h4>
 
 > 8) Role - integer. Value "7" means teacher's role
 
<h4>Example of usage</h4>

> We need to know about user's files

<h4>Steps:</h4>

1. Authorization
2. Get user's files

1. Sending POST request to http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/api with data:{method:"signin", params:{"Alexey", "1234"}}
2. Catching response from API and obtaining <b>token</b>
3. Using token for method [get_files] - http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/api?method=get_files&token=91c26f0fec6f834d928fcc644ef8532849803f77. We'll receive json with status(1-ok,0-error,...), description(Text for human) and json array with file's info

========

<h4>Small sample of code</h4>

	```javascript
	
	$(function(){
		$.ajax({
			type: "POST",
			url: "http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/api",
			dataType: "json",
			data: { method:"signin", params:{login:"alexey@oxford.com", pass:"mypass123"}},
			success: function(data){
				// Using data
			}
		});
	});
	
	```
	
<h4>Sample of uploading file with custom title and description</h4>

	```html
	
	<form enctype="multipart/form-data" method="post" id="formaFile">
		<input type="file" name="novp_file" size="20" />
		<input type="hidden" name="method" value="upload_file">
		<input type="hidden" name="title" value="Holidays">
		<input type="hidden" name="description" value="My amazing holidays">
		<input type="hidden" name="token" value="ed0a8b8c12bea7d4a64b9afb38332646457fd693">
		<input type="submit" value="upload_file">
	</form>		
	
	```
	
	```javascript
	
	$("#formaFile").submit(function(e) {
			var formData = new FormData($("#formaFile")[0]);
			$.ajax({
				type: "POST",
				url: "http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com/app/api",
				processData: false,
  				contentType: false,
				data: formData,				
				success: function(data){
					console.log(data); // Check response data
				}
		});
		e.preventDefault();			
	});
	
	```

	
#Chat API Docs(NodeJS Socket)

<h3> Socket connection </h3>

1. Enable socket script to site where API will be used:

	
	```javascript

	<script type="text/javascript" src="http://[domain]:8081/socket.io/socket.io.js"></script> 

	```

	> [Domain] - ec2-54-68-182-31.us-west-2.compute.amazonaws.com


2. Obtain token with authorization previous method (signin):


	```javascript

	var token = data.token;
		 
	```

	> where [data] json response for method signin
	
	
3. Create socket object (using token):


	```javascript

	var socket = io('http://ec2-54-68-182-31.us-west-2.compute.amazonaws.com:8081/api?token=[token]');

	```


4. You should specify the room (rooms):

	
	```javascript
	
    var room = "test_room"; (*example*)
	
	```
	

5. For joining to that room send message to socket(join_room):
	
	
	```javascript
	
	socket.emit('join_room', {room:room});
	
	```


6. We can listen log of our activity("notifications"):
	
	
	```javascript
	
	socket.on('notifications', function(data){
		$("#responses").html("<p>Status: "+data.status+". "+data.description+"</p>");
	});
	
	```	
	
	> In response JSON we can get "status" parameter and "description" parameter so we can react to.
	> Also response data contains "request_type" field it helps defined response type. For now API supports request_type with values: "operation" and "invite"


7. To handle with "sending messages" operation just send message to socket(send_message) using token and room:
	- For example we want to send some text from "keypress" event
	
	
	
	```javascript
	
	$("#messager input").keypress(function(e){
		if(e.which == 13){
			var message = $(this).val();
			socket.emit("send_message", {room:room, msg:message});
		}
	});
	
	```


8. Also we should listen and wait for new messages in our room
	
	
	```javascript
	
	socket.on('get_messages', function(data){
		var message_display = $("#messager ul");
		$.each(data, function(i, val){
			message_display.append("<li><p><img [src]='"+val.avatar+"'></p><p>"+val.name+"</p><p>"+val.msg+"</p></li>");			
		});
	});
	
	```
	
	
	> Our response data is array of messages(objects)
	> if we will join multiple rooms we should sort our response data, so check val.path that contains room;

	
<h3>Invitation for chat</h3>

1. Send message to socket for triggering invitation event <b>('invite_to_chat')</b>
	
	```javascript
	
	var uid = //;
	socket.emit('invite_to_chat', {uid:uid});
	
	```

	> On this step we should identify the user we want to speak with ( for now it is "user id" )
	
2. Users should listen for possible invitations so developer who uses API should listen and catch notifications messages <b>('notifications')</b>.
   
	```javascript
	
	socket.on('notifications', function(data){
		// For now only invite to chat
		if(data.request_type == 'invite'){
			// So we need to react for such invitation
		}else if(data.request_type == 'operation'){
			// This data is just responses of different operations (display - optional)
		}
	});
	
	```
	
	> It is important to understand and handle the type of response notifications!
	> "request_type" for now contains only 2 value "invite" or "operations"
	> Developers should handle invite case and notify users with some Prompt window or Confirm, or ...

3. On this step user received a notification for inviting. So after he'll accept it we should send a message about successful accepting <b>('accept_invitaion')</b>

	```javascript
		
		...
		if(data.request_type == 'invite'){
			var reponse = confirm('You have recieved an invitation from  '+data.name + ". Accept?");
			if(reponse){
				// Accept invitation
				socket.emit('accept_invitaion', {room: data.room}); // 
			}
		}
		...
		
	```
	
	> Response data contains field - "room" - <b>data.room<b>, which we will use for sending messages
	
4. The form of sending messages is still the same

	```javascript
	
	...
	socket.emit("send_message", {room:room, msg:message});
	...
	
	```
	
<h3>WOD Chat specialities</h3>

1. For Wod chat box(room) you should specified additional prefix(<b>"box_"</b>)

	```javascript
	
	...
	var room = "box_" + box_id;
	...
	
	```
	
	> <i>box_id</i> - it's just WOD chat room

<h3>Chat Statistics</h3>

1. Getting online info in specific "room" (<b>get_room_users</b>):

	```javascript
	
	...
	socket.emit('get_room_users', {room: "Room name"});
	...
	
	```
	
2. Catching response of JSON users data in <b>"notification"</b> listener specified by <i>"request_type = users_in_room"</i>
	
	```javascript
	
	...
	else if(data.request_type == 'users_in_room'){
		var users = data.users; 
	}
	...
	
	```
	
	> Object "data.users" contains users.
	> We can count users in specific room or getting other info.
	
3. Also developers have access for knowing total amount of messages in specific room (<b>count_room_records</b>):

	```javascript
	
	...
	socket.emit('count_room_records', {room:room});
	...
	
	```
	
	> Getting response from <b>"notification"</b> listener in defined by <i>"request_type=room_records"</i>
	> Amount of messages we can find in <b>data.count</b>