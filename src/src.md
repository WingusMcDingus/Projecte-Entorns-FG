# Codi Dummy
extends CharacterBody3D

const SPEED = 4.0

@onready var ap = $AnimationPlayer 

 --- NUEVOS ESTADOS (NEW STATES) ---
var is_attacking = false
var is_hurt = false
var is_knocked_down = false

func _physics_process(delta):
	# 1. Gravedad
	if not is_on_floor():
		velocity += get_gravity() * delta

	# 2. Máquina de estados para el movimiento
	if is_hurt or is_knocked_down:
		# Si está herido o en el piso, lo frenamos
		velocity.x = move_toward(velocity.x, 0, SPEED)
	elif not is_attacking:
		# Comportamiento normal (como es un dummy, solo se queda quieto)
		velocity.x = move_toward(velocity.x, 0, SPEED)
		play_animation("HornFS") # Idle
	else:
		# Si por alguna razón está atacando, lo frenamos
		velocity.x = move_toward(velocity.x, 0, SPEED)

	# 3. Bloquear eje Z
	velocity.z = 0
	position.z = 0
	
	move_and_slide()

 --- FUNCIÓN PARA RECIBIR GOLPES ---
# Llama a esta función enviando el tipo de ataque (ej: "light", "heavy")
func take_hit(attack_type: String):
	# Si ya está en el suelo, ignoramos nuevos golpes para no reiniciar la animación
	if is_knocked_down:
		return
		
	# Cancelamos cualquier ataque que estuviera haciendo
	is_attacking = false
	
	if attack_type == "light" or attack_type == "low":
		is_hurt = true
		play_anim_no_loop("getHit")
	elif attack_type == "heavy" or attack_type == "overhead":
		is_knocked_down = true
		play_anim_no_loop("KnockedDown")

 Helper function to play an animation and force it to NOT loop
func play_anim_no_loop(anim_name):
	var anim = ap.get_animation(anim_name)
	if anim:
		anim.loop_mode = Animation.LOOP_NONE
	ap.play(anim_name)

func play_animation(anim_name):
	if ap.current_animation != anim_name:
		ap.play(anim_name)

 --- CONTROLADOR DE FIN DE ANIMACIONES ---
func _on_animation_player_animation_finished(anim_name: StringName) -> void:
	# Si terminó de atacar
	if is_attacking:
		is_attacking = false
		
	# Si terminó su animación de dolor leve
	elif anim_name == "getHit": 
		is_hurt = false
		
	# Si terminó de caer al piso, inmediatamente reproducimos la de levantarse
	elif anim_name == "KnockedDown": 
		play_anim_no_loop("StandUP") 
		
	# Si terminó de levantarse, le devolvemos el control
	elif anim_name == "StandUP": 
		is_knocked_down = false
		
		# --- TEMPORARY TESTING CONTROLS ---
func _input(event):
	if Input.is_key_pressed(KEY_1):
		take_hit("light") # Triggers getHit
	elif Input.is_key_pressed(KEY_2):
		take_hit("heavy") # Triggers KnockedDown and then StandUP

# Codi Jugador
extends CharacterBody3D

const SPEED = 3.0

@onready var ap = $AnimationPlayer 

var is_attacking = false
@export var opponent: CharacterBody3D

func _input(event):
	# Si ya está atacando, ignoramos otros botones para que no corte la animación a medias
	if is_attacking:
		return
		
	if Input.is_action_just_pressed("p1_attackL"):
		start_attack("HornJab")
	elif Input.is_action_just_pressed("p1_attackH"):
		start_attack("HornHeavy")
	elif Input.is_action_just_pressed("p1_attackLW"):
		start_attack("HornLow")
	elif Input.is_action_just_pressed("p1_attackOH"):
		start_attack("HornOverhead")

func _physics_process(delta):
	# 1. Gravedad
	if not is_on_floor():
		velocity += get_gravity() * delta

	# 2. Movimiento
	if not is_attacking:
		var input_dir = Input.get_axis("p1_left", "p1_right")
		if input_dir > 0:
			velocity.x = input_dir * SPEED
			play_animation("walkForw")
		elif input_dir < 0:
			velocity.x = input_dir * SPEED
			play_animation("walkBack")
		else:
			velocity.x = move_toward(velocity.x, 0, SPEED)
			play_animation("HornFS") # Idle
	else:
		# Comportamiento mientras ESTÁ atacando
		if ap.current_animation == "HornHeavy":
			# Empujamos al personaje hacia adelante solo mientras dura la animación
			velocity.x = 0.5
		else:
			# Para el resto de los ataques, lo frenamos en seco
			velocity.x = move_toward(velocity.x, 0, SPEED)

	# 3. Bloquear eje Z
	velocity.z = 0
	position.z = 0
	
	move_and_slide()

func play_animation(anim_name):
	if ap.current_animation != anim_name:
		ap.play(anim_name)

func start_attack(anim_name):
	is_attacking = true
	
	# Apagar el loop
	var anim = ap.get_animation(anim_name)
	if anim:
		anim.loop_mode = Animation.LOOP_NONE
		
	ap.play(anim_name)
	
	# --- DETECTOR DE GOLPES CON RETRASO (DELAY) ---
	if opponent != null:
		var distance = global_position.distance_to(opponent.global_position)
		
		# Si está lo suficientemente cerca
		if distance < 2.5: 
			
			if anim_name == "HornLow":
				# Espera 0.3 segundos para sincronizar con la animación de la patada
				await get_tree().create_timer(0.3).timeout
				# Verificamos de nuevo la distancia por si el enemigo se movió
				if global_position.distance_to(opponent.global_position) < 2.5:
					opponent.take_hit("light")
					
			elif anim_name == "HornJab":
				# El jab es muy rápido, lo dejamos instantáneo (o añade un timer muy corto como 0.1)
				opponent.take_hit("light")
				
			elif anim_name == "HornHeavy" or anim_name == "HornOverhead":
				# También puedes añadir un 'await' aquí si los ataques pesados tardan en golpear
				# await get_tree().create_timer(0.4).timeout
				opponent.take_hit("heavy")

func _on_animation_player_animation_finished(anim_name: StringName) -> void:
	if is_attacking:
		is_attacking = false
