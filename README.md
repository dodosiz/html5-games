<!doctype html>
<html>	
	<head>
		<meta charset="UTF-8">
		<meta name="viewport" content="width=device-width, initial-scale=1">
		<title>html5 game</title>
		<link rel="stylesheet" href="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
		<script src="https://ajax.googleapis.com/ajax/libs/jquery/1.12.4/jquery.min.js"></script>
		<script src="http://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
		<style>
			html {
			width: 100%;
			height: 100%;
			clip: auto;
			position: absolute;
			overflow: hidden;
			}
			button {
				background-color: #4CAF50; /* Green */
				border: none;
				color: white;
				padding: 15px 32px;
				text-align: center;
				text-decoration: none;
				display: inline-block;
				font-size: 16px;
				margin: 60px 30px;
			}
			p{
				margin-top: 20px;
			}
		</style>
		<script>
		$(document).ready(function(){
			$("canvas").hide();
			$("#show").click(function(){
				$("canvas").show();
				$("#show").hide();
			});
		});
		</script>
	</head>
	<body>
		<div class="row">
			<div class="col-sm-5">
				<button id="show">Play!</button>
				<canvas id="ctx" width="500" height="500" style="border:1px solid #000000;"></canvas>
			</div>
			<div class="col-sm-7">
				<p>Directions:
					<ul>
						<li>Move up, down, left, right with arrows</li>
						<li>Shoot with space</li>
						<li>Collect the purple boxes to gain attack speed</li>
						<li>Collect the orange boxes to increase your score</li>
						<li>Avoid the red enemies</li>
					</ul>
				</p>
			</div>
		</div>
		<script>
			var ctx = document.getElementById("ctx").getContext("2d");
			ctx.font = "30px Arial";

			var HEIGHT = 500;
			var WIDTH = 500;
			var timeStarted = Date.now();

			var frameCount = 0;

			var score = 0;

			var player = {
				x:50,
				speedX:30,
				y:40,
				speedY:5,
				name:'P',
				hp:10,
				width:40,
				height:20,
				color:'green',
				attack: 1,
				attackCounter:0,
				pressingDown:false,
				pressingUp:false,
				pressingLeft:false,
				pressingRight:false
			};

			var enemyList = {};
			var upgradeList = {};
			var bulletList = {};

			getDistance = function(entity1,entity2){
				var vx = entity1.x - entity2.x;
				var vy = entity1.y - entity2.y;
				return Math.sqrt(vx*vx+vy*vy);
			}

			testCollision = function(entity1, entity2){
				var rect1 = {
					x: entity1.x - entity1.width/2,
					y: entity1.y - entity1.height/2,
					width: entity1.width,
					height: entity1.height
				}
				
				var rect2 = {
					x: entity2.x - entity2.width/2,
					y: entity2.y - entity2.height/2,
					width: entity2.width,
					height: entity2.height
				}
				
				return testCollisionRect(rect1,rect2);
			}

			Enemy = function(id,x,speedX,y,speedY,width,height){
				var enemy = {
					x:x,
					speedX:speedX,
					y:y,
					speedY:speedY,
					id:id,
					width:width,
					height:height,
					color:'red'
				};
				enemyList[id] = enemy;
			}

			Upgrade = function(id,x,speedX,y,speedY,width,height,category,color){
				var upgrade = {
					x:x,
					speedX:speedX,
					y:y,
					speedY:speedY,
					id:id,
					width:width,
					height:height,
					color:color,
					category:category,
					timer:0
				};
				upgradeList[id] = upgrade;
			}

			randomUpgrade = function(){
				var x = Math.random()*WIDTH;
				var y = Math.random()*HEIGHT;
				var height = 10;
				var width = 10;
				var id = Math.random();
				var speedX = 0;
				var speedY = 0;
				
				if(Math.random()<0.5){
					var category = 'score';
					var color = 'orange';
				}else{
					var category = 'attack';
					var color = 'purple';
				}
				
				Upgrade(id,x,speedX,y,speedY,width,height,category,color);
			}

			Bullet = function(id,x,speedX,y,speedY,width,height){
				var bullet = {
					x:x,
					speedX:speedX,
					y:y,
					speedY:speedY,
					id:id,
					width:width,
					height:height,
					color:'black',
					timer:0
				};
				bulletList[id] = bullet;
			}

			randomBullet = function(){
				var x = player.x;
				var y = player.y;
				var height = 10;
				var width = 10;
				var id = Math.random();
				
				var angle = Math.random()*2*Math.PI;
				var speedX = Math.cos(angle)*5;
				var speedY = Math.sin(angle)*5;
				Bullet(id,x,speedX,y,speedY,width,height);
			}

			document.onmousemove = function(mouse){
				/*var mouseX = mouse.clientX - document.getElementById('ctx').getBoundingClientRect().left;
				var mouseY = mouse.clientY - document.getElementById('ctx').getBoundingClientRect().top;
				
				if(mouseX<player.width/2)
					mouseX = player.width/2;
					
				if(mouseX>WIDTH - player.width/2)
					mouseX = WIDTH - player.width/2;
					
				if(mouseY<player.height/2)
					mouseY = player.height/2;
					
				if(mouseY>HEIGHT - player.height/2)
					mouseY = HEIGHT - player.height/2;
				
				player.x = mouseX;
				player.y = mouseY;*/
			}

			updateEntity = function(something){
				updateEntityPosition(something);
				drawEntity(something);
			}

			testCollisionRect = function(rect1, rect2){
				return rect1.x <= rect2.x + rect2.width
					&& rect2.x <= rect1.x + rect1.width
					&& rect1.y <= rect2.y + rect2.height
					&& rect2.y <= rect1.y + rect1.height;
			}

			drawEntity = function(something){
				ctx.save();
				ctx.fillStyle = something.color;
				ctx.fillRect(something.x-something.width/2,something.y-something.height/2,something.width,something.height);
				ctx.restore();
			}

			updateEntityPosition = function(something){
				something.x += something.speedX;
				something.y += something.speedY;
				if(something.x > WIDTH || something.x < 0){
					something.speedX = -something.speedX;
				}
				
				if(something.y > HEIGHT || something.y < 0){
					something.speedY = -something.speedY;
				}
			}

			document.onclick = function(mouse){
				/*if(player.attackCounter > 25){
					randomBullet();
					player.attackCounter = 0;
				}*/
			}

			document.onkeydown = function(event){
				if(event.keyCode == 39){
					player.pressingRight = true;
				}else if(event.keyCode == 38){
					player.pressingUp = true;
				}else if(event.keyCode == 37){
					player.pressingLeft = true;
				}else if(event.keyCode == 40){
					player.pressingDown = true;
				}else if(event.keyCode == 32){
					if(player.attackCounter > 25){
					randomBullet();
					player.attackCounter = 0;
				}
				}
			}

			document.onkeyup = function(event){
				if(event.keyCode == 39){
					player.pressingRight = false;
				}else if(event.keyCode == 38){
					player.pressingUp = false;
				}else if(event.keyCode == 37){
					player.pressingLeft = false;
				}else if(event.keyCode == 40){
					player.pressingDown = false;
				}else if(event.keyCode == 32){
				
				}
			}

			updatePlayerPosition = function(){
				if(player.pressingRight)
					player.x += 10;
				if(player.pressingLeft)
					player.x -= 10;
				if(player.pressingUp)
					player.y -= 10;
				if(player.pressingDown)
					player.y += 10;
				//is position valid
				if(player.x<player.width/2)
					player.x = player.width/2;
				if(player.x>WIDTH - player.width/2)
					player.x = WIDTH - player.width/2;
				if(player.y<player.height/2)
					player.y = player.height/2;
				if(player.y>HEIGHT - player.height/2)
					player.y = HEIGHT - player.height/2;
			}

			update = function(){
				ctx.clearRect(0,0,WIDTH,HEIGHT);
				frameCount++;
				score++;
				
				if(frameCount % 100 == 0)
					randomEnemy();
				
				if(frameCount % 75 == 0)
					randomUpgrade();
				
				player.attackCounter += player.attack;
				
				for(var key in bulletList){
					updateEntity(bulletList[key]);
					bulletList[key].timer++;
					if(bulletList[key].timer>100){
						delete bulletList[key];
						continue;
					}
					
					for(var key2 in enemyList){
						var isColliding = testCollisionRect(bulletList[key],enemyList[key2]);
						if(isColliding){
							delete bulletList[key];
							delete enemyList[key2];
							break;
						}
					}
					
				}
				
				for(var key in upgradeList){
					updateEntity(upgradeList[key]);
					var isColliding = testCollisionRect(player,upgradeList[key]);
					upgradeList[key].timer++;
					if(upgradeList[key].timer>200){
						delete upgradeList[key];
						continue;
					}
					if(isColliding){
						if(upgradeList[key].category == 'score')
							score += 1000;
						if(upgradeList[key].category == 'attack')
							player.attack += 3;
						delete upgradeList[key];
					}
				}
				
				for(var key in enemyList){
					updateEntity(enemyList[key]);
					var isColliding = testCollision(player,enemyList[key]);
					if(isColliding){
						var timeSurvived = Date.now() - timeStarted;
						player.hp = player.hp - 1;
					}
				}
				if(player.hp <= 0){
					ctx.clearRect(0,0,WIDTH,HEIGHT);
					startNewGame();
				}
				updatePlayerPosition();
				drawEntity(player);
				ctx.fillText(player.hp + " Hp",0,30);
				ctx.fillText("Score: " + score,200,30);
			}

			randomEnemy = function(){
				var x = Math.random()*WIDTH;
				var y = Math.random()*HEIGHT;
				var height = 10 + Math.random()*30;
				var width = 10 + Math.random()*30;
				var id = Math.random();
				var speedX = 5 + Math.random();
				var speedY = 5 + Math.random();
				Enemy(id,x,speedX,y,speedY,width,height);
			}

			startNewGame = function(){
				player.hp = 10;
				timeStarted = Date.now();
				frameCount = 0;
				enemyList = {};
				randomEnemy();
				randomEnemy();
				randomEnemy();
				score = 0;
				upgradeList = {};
				bulletList = {};
			}

			startNewGame();

			setInterval(update,40);

		</script>
	</body>
</html>