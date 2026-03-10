<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>CONECTA Cotizador Inteligente</title>

<style>

body{
background:#0b0b0b;
font-family:Arial,Helvetica,sans-serif;
color:white;
margin:0;
padding:20px;
}

.container{
max-width:520px;
margin:auto;
background:#121212;
padding:20px;
border-radius:16px;
box-shadow:0 0 20px rgba(0,0,0,0.5);
}

h2{
margin-top:0;
color:#4cc3ff;
}

label{
display:block;
margin-top:12px;
font-size:14px;
color:#ccc;
}

input{
width:100%;
padding:10px;
margin-top:5px;
border-radius:10px;
border:none;
background:#1e1e1e;
color:white;
font-size:16px;
}

.row{
display:flex;
gap:10px;
}

.row input{
flex:1;
}

button{
width:100%;
padding:14px;
margin-top:10px;
border:none;
border-radius:12px;
font-size:16px;
cursor:pointer;
}

.btn-blue{background:#2d8bd6;color:white;}
.btn-green{background:#19c463;color:black;}
.btn-gray{background:#444;color:white;}

.result{
margin-top:15px;
font-size:20px;
color:#40e0ff;
text-align:center;
}

</style>
</head>

<body>

<div class="container">

<h2>CONECTA • Cotizador Inteligente</h2>

<label>Origen</label>
<input id="origen" placeholder="Ej: Toreo Parque Central">

<label>Destino</label>
<input id="destino" placeholder="Ej: Paseo de la Reforma">

<h2>Tarifas y umbrales</h2>

<div class="row">
<div>
<label>$ por km</label>
<input id="tarifaKm" type="number" value="8">
</div>

<div>
<label>$ por hora</label>
<input id="tarifaHora" type="number" value="170">
</div>
</div>

<label>Mínimo cobro</label>
<input id="minimo" type="number" value="50">

<label>
<input type="checkbox" id="trafico" checked>
Tarifa escalonada por tráfico
</label>

<h2>Calcular</h2>

<button class="btn-blue" onclick="calcular()">Calcular</button>
<button class="btn-gray" onclick="prueba()">Prueba CDMX → Toluca</button>
<button class="btn-green" onclick="abrirWaze()">Abrir en Waze</button>
<button class="btn-gray" onclick="limpiar()">Limpiar</button>
<button class="btn-green" onclick="enviarWhats()">Enviar cotización por WhatsApp</button>

<div id="resultado" class="result">
Total estimado: $0 MXN
</div>

</div>

<script>

const telefono = "5619508426"

async function geocode(direccion){

let url="https://nominatim.openstreetmap.org/search?format=json&q="+encodeURIComponent(direccion)

let r=await fetch(url)
let data=await r.json()

if(data.length==0) throw "Dirección no encontrada"

return{
lat:parseFloat(data[0].lat),
lon:parseFloat(data[0].lon)
}

}

function distancia(lat1,lon1,lat2,lon2){

let R=6371

let dLat=(lat2-lat1)*Math.PI/180
let dLon=(lon2-lon1)*Math.PI/180

let a=
Math.sin(dLat/2)*Math.sin(dLat/2)+
Math.cos(lat1*Math.PI/180)*Math.cos(lat2*Math.PI/180)*
Math.sin(dLon/2)*Math.sin(dLon/2)

let c=2*Math.atan2(Math.sqrt(a),Math.sqrt(1-a))

return R*c

}

async function calcular(){

let origen=document.getElementById("origen").value
let destino=document.getElementById("destino").value

if(!origen || !destino){
alert("Ingresa origen y destino")
return
}

try{

let coord1=await geocode(origen)
let coord2=await geocode(destino)

let km=distancia(coord1.lat,coord1.lon,coord2.lat,coord2.lon)

km=km*1.35

if(km>120){
alert("Distancia demasiado grande o error en dirección")
return
}

let tarifaKm=parseFloat(document.getElementById("tarifaKm").value)
let tarifaHora=parseFloat(document.getElementById("tarifaHora").value)
let minimo=parseFloat(document.getElementById("minimo").value)
let trafico=document.getElementById("trafico").checked

let velocidad=28

let horas=km/velocidad

let precioKm=km*tarifaKm
let precioTiempo=horas*tarifaHora

let total=precioKm+precioTiempo

if(trafico) total*=1.25

if(total<minimo) total=minimo

total=Math.round(total)

window.cotizacionActual={
origen,destino,km,total
}

document.getElementById("resultado").innerHTML=
"Total estimado: $"+total+" MXN<br>"+
"Distancia aprox: "+km.toFixed(1)+" km"

}

catch(e){

alert("No se pudo calcular: "+e)

}

}

function prueba(){

document.getElementById("origen").value="Ciudad de México"
document.getElementById("destino").value="Toluca"

}

function abrirWaze(){

let destino=document.getElementById("destino").value

let url="https://waze.com/ul?q="+encodeURIComponent(destino)

window.open(url,"_blank")

}

function limpiar(){

document.getElementById("origen").value=""
document.getElementById("destino").value=""
document.getElementById("resultado").innerHTML="Total estimado: $0 MXN"

}

function enviarWhats(){

if(!window.cotizacionActual){
alert("Primero calcula la cotización")
return
}

let c=window.cotizacionActual

let msg=
"🚗 Cotización CONECTA%0A"+
"Origen: "+c.origen+"%0A"+
"Destino: "+c.destino+"%0A"+
"Distancia aprox: "+c.km.toFixed(1)+" km%0A"+
"Total estimado: $"+c.total+" MXN"

let link="https://wa.me/"+telefono+"?text="+msg

window.open(link,"_blank")

}

</script>

</body>
</html>
 
