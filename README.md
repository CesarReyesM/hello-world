//variables y selectores
const formulario = document.querySelector('#agregar-gasto');
const gastoListado = document.querySelector('#gastos ul');
// eventos
eventListener();

function eventListener(){
    document.addEventListener('DOMContentLoaded', preguntarPresupuesto);}

    formulario.addEventListener('submit', agregarGasto);




// clases
class Presupuesto {
  constructor (presupuesto){
    this.presupuesto = Number(presupuesto);
    this.restante = Number(presupuesto);
    this.gastos = [];
  }
  nuevoGasto(gasto){
    this.gastos =[...this.gastos, gasto]
    this.calcularRestante();
  }
  calcularRestante(){
    const gastado = this.gastos.reduce((total,gasto)=>total + gasto.cantidad, 0);
    this.restante = this.presupuesto - gastado;
  }
  eliminarGasto(id){
    this.gastos = this.gastos.filter(gasto => gasto.id !== id);
    this.calcularRestante();
  }
}

class UI{
  insertarPresupuesto(cantidad){
    //extrayendo los valores
    const {presupuesto, restante} = cantidad;
    // agregando al html
    document.querySelector('#total').textContent = presupuesto;
    document.querySelector('#restante').textContent = restante;
  }
  imprimirAlerta(mensaje,tipo){
    // crear el div alerta
    const divMensaje = document.createElement('div');
    divMensaje.classList.add('text-center', 'alert');

    if(tipo === 'error'){
      divMensaje.classList.add('alert-danger');
    }else{
      divMensaje.classList.add('alert-success')
    }
    // Mensaje de error
    divMensaje.textContent= mensaje
    // Insertar en el HTML
    document.querySelector('.primario').insertBefore(divMensaje, formulario);
    // Quitar el mensaje
    setTimeout(()=>{
      divMensaje.remove();
    },3000)
  }
  //agregando el gasto al HTML
  mostrarGastos(gastos){
    this.limpiarHTML(); // elimina el HTML previo
    // Iterar sobre los gastos
    gastos.forEach(gasto =>{
      const {nombre, cantidad, id}= gasto;
      // crear un LI
      const nuevoGasto = document.createElement('li');
      nuevoGasto.className = 'list-group-item d-flex justify-content-between align-items-center';
      nuevoGasto.dataset.id = id // Forma actual para hacerlo

      // Agregar el HTML del gasto
     
      
      nuevoGasto.innerHTML = 
      `${nombre} <span class= 'badge badge-primary badge-pill'>$${cantidad}</span>`
      // Boton para borrar el gasto

      const btnBorrar = document.createElement('button');
      btnBorrar.classList.add('btn', 'btn-danger', 'borrar-gasto');
      btnBorrar.innerHTML = 'Borrar &times;'

      btnBorrar.onclick = ()=>{
        eliminarGasto(id)
      }
      nuevoGasto.appendChild(btnBorrar);
     
      // Agregar al HTML
      gastoListado.appendChild(nuevoGasto);
      
     
    })
  }
  limpiarHTML(){
    while(gastoListado.firstChild){
      gastoListado.removeChild(gastoListado.firstChild);
    }
  }

  actualizarRestante(restante){
    document.querySelector('#restante').textContent = restante;
  }
  comprobarPresupuesto(presupuestoObj){
    const {presupuesto, restante} = presupuestoObj;

    const restanteDiv = document.querySelector('.restante');
 
    // comprobar 25%
    if((presupuesto /4)>= restante){
      restanteDiv.classList.remove('alert-success', 'alert-warning');
      restanteDiv.classList.add('alert-danger');
    }else if ((presupuesto /2)>= restante){
      restanteDiv.classList.remove('alert-success');
      restanteDiv.classList.add('alert-warning');
    }else{
      restanteDiv.classList.remove('alert-danger', 'alert-warning');
      restanteDiv.classList.add('alert-success')
    }



    // Si el total es menor a Cero 
    if(restante <= 0){
      ui.imprimirAlerta('El presupuesto se ha agotado', 'error');

      formulario.querySelector('button[type="submit"]').disabled = true;
    }
  }



}
// instancia
const ui = new UI();
let presupuesto;
// funciones
function preguntarPresupuesto (){
    const presupuestoUsuario = prompt('Cual es tu presupuesto?');
    //console.log(Number(presupuestoUsuario));
    if(presupuestoUsuario ===''|| presupuestoUsuario === null||isNaN(presupuestoUsuario)||
    presupuestoUsuario <=0){
        window.location.reload();
    }
    presupuesto = new Presupuesto(presupuestoUsuario);

    console.log(presupuesto);

    ui.insertarPresupuesto(presupuesto);
}

//añade gasto
function agregarGasto(e){
  e.preventDefault();

  
  // leer los datos del formulario
  const nombre = document.querySelector('#gasto').value;

  const cantidad = Number(document.querySelector('#cantidad').value);
  // Validar que no esten vacios
  if(nombre===''|| cantidad ===''){
    ui.imprimirAlerta('Ambos campos son obligatorios', 'error');
    return;
  }else if(cantidad <= 0){
    ui.imprimirAlerta('No puedes tener gastos negativos', 'error')
    return;
  }else if(isNaN(cantidad)){
    ui.imprimirAlerta('Solo puedes agregar numero en Cantidad', 'error')
    return;
  }
  // Generar un objeto con el gasto
  const gasto = {nombre, cantidad, id: Date.now()};
  // es lo mismo que escrirlo asi 
  //{
  // nombre : nombre, 
  // cantidad: cantidad, etc
  //}

  // Añade un nuevo Gasto
  presupuesto.nuevoGasto(gasto)
  // mensaje de todo bien
  ui.imprimirAlerta('Gasto agregado Correctamente');
 
  // Imprimir los gastos
  const {gastos, restante} = presupuesto;
  ui.mostrarGastos(gastos);

  ui.actualizarRestante(restante);

  ui.comprobarPresupuesto(presupuesto);

  // reinicia formulario
  formulario.reset();

}

function eliminarGasto(id){
  // este eliminar del objetos los gastos
  presupuesto.eliminarGasto(id)
  // eliminar los gastos del html
  const {gastos, restante} = presupuesto;
  ui.mostrarGastos(gastos);

  ui.actualizarRestante(restante);

  ui.comprobarPresupuesto(presupuesto);
}
