// Clic al botón de consulta para llamar a la función fetchAvailability
document.getElementById('boton_de_consulta').addEventListener('click', fetchAvailability);

// Formateo de fecha en formato ISO
function getFormattedDate(date) {
  return date.toISOString().split('.')[0] + 'Z';
}

// Obtención de las fechas de inicio y fin (72 horas después del momento actual)
function getStartAndEndTimes() {
  const now = new Date();
  const start = new Date(now.getTime());
  const end = new Date(now.getTime() + 72 * 60 * 60 * 1000); // 72 horas después
  return {
    start: getFormattedDate(start),
    end: getFormattedDate(end)
  };
}

// Consultar la disponibilidad del producto
async function fetchAvailability() {
  const apiKey = 'yoJYongi4V4m0S4LClubdyiu5nq6VIpxazcFaghi';
  const url = 'https://api.xandar.instaleap.io/jobs/availability/v2';
  const valorPago = document.getElementById('valor').value.trim();

  // Validar que se haya ingresado un valor de pago válido
  if (!valorPago) {
    alert('Por favor, ingresa un valor de pago válido.');
    return;
  }

  // Obtener las fechas de inicio y fin formateadas
  const { start, end } = getStartAndEndTimes();

  try {
    // Realizar una solicitud POST a la API para consultar la disponibilidad
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'x-api-key': apiKey
      },
      body: JSON.stringify({
        origin: {
          name: '101_FS',
          address: '101_FS',
          latitude: 4.76241364477749,
          longitude: -74.04646166418708
        },
        destination: {
          name: 'Destino',
          address: 'Prueba',
          latitude: 4.642712338208402,
          longitude: -74.07520878834313
        },
        currency_code: 'USD',
        start: start,
        end: end,
        slot_size: 60,
        operational_models_priority: ['FULL_SERVICE'],
        store_reference: '101_FS',
        job_items: [
          {
            id: 'Item1',
            name: 'Item1',
            unit: 'KG',
            sub_unit: 'kg - KG - kG - Kg',
            quantity: 1,
            sub_quantity: 1,
            price: parseFloat(valorPago) // Convertir el valor de pago a número
          }
        ],
        fallback: true
      })
    });

    // Notificación de errores si la respuesta no es exitosa
    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Error al consultar disponibilidad: ${errorText}`);
    }

    // Obtener los datos de la respuesta como JSON
    const data = await response.json();
    console.log('Respuesta de disponibilidad:', data);

    // Renderizar los slots de disponibilidad en la interfaz de usuario
    renderAvailabilitySlots(data);
  } catch (error) {
    console.error('Error:', error);
    alert(`Hubo un error al consultar la disponibilidad. Por favor, intenta de nuevo. Error: ${error.message}`);
  }
}

// Formatear fecha del slot seleccionado en formato DD/MM/AAAA hh:mm:ss para mostrarlo en el front
function formatDate(dateString) {
  const date = new Date(dateString);
  const options = { day: '2-digit', month: '2-digit', year: 'numeric', hour: '2-digit', minute: '2-digit', second: '2-digit' };
  return date.toLocaleString('es-ES', options).replace(',', ''); // Formato DD/MM/AAAA hh:mm:ss
}

// Renderizar los slots de disponibilidad en la interfaz de usuario
function renderAvailabilitySlots(data) {
  const slotsContainer = document.getElementById('slots_de_disponibilidad');
  slotsContainer.innerHTML = '';

  const slots = Array.isArray(data) ? data : [];

  // Crear elementos para cada slot de disponibilidad y añadirlos al contenedor
  slots.forEach(slot => {
    const slotElement = document.createElement('div');
    slotElement.classList.add('slot');
    slotElement.textContent = `De: ${formatDate(slot.from)}, a: ${formatDate(slot.to)}`;
    slotElement.addEventListener('click', () => {
      createOrder(slot.id); // Llamar a la función para crear una orden al hacer clic en el slot
    });
    slotsContainer.appendChild(slotElement);
  });
}

// Función para crear una orden con el slot seleccionado
async function createOrder(slotId) {
  const apiKey = 'yoJYongi4V4m0S4LClubdyiu5nq6VIpxazcFaghi';
  const url = 'https://api.xandar.instaleap.io/jobs';
  const clientReference = `client_${new Date().getTime()}`;

  try {
    // Realizar una solicitud POST a la API para crear una orden
    const response = await fetch(url, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Accept': 'application/json',
        'x-api-key': apiKey
      },
      body: JSON.stringify({
        recipient: {
          name: 'Prueba',
          email: 'camilojb2@gmail.com',
          phone_number: '3108021173'
        },
        payment_info: {
          prices: {
            order_value: parseFloat(document.getElementById('valor').value.trim()) // Convertir el valor de pago a número
          },
          payment: {
            method: 'CASH'
          },
          currency_code: 'USD'
        },
        add_delivery_code: true,
        contact_less: {
          comment: 'LeaveAtTheDoor',
          cash_receiver: 'asdfasdaf',
          phone_number: '213313213212'
        },
        slot_id: slotId,
        client_reference: clientReference,
        job_items: [
          {
            id: 'Item1',
            name: 'Item1',
            unit: 'KG',
            sub_unit: 'kg - KG - kG - Kg',
            quantity: 1,
            sub_quantity: 1,
            price: parseFloat(document.getElementById('valor').value.trim()) // Convertir el valor de pago a número
          }
        ]
      })
    });

    // Notificación de errores si la respuesta no es exitosa
    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Error al crear la orden: ${errorText}`);
    }

    // Obtener los datos de la respuesta como JSON
    const responseData = await response.json();
    console.log('Orden creada:', responseData);
    alert('Orden creada exitosamente.');

    // Obtener detalles del trabajo (orden) creado
    const jobId = responseData.job_id; 
    if (jobId) {
      await fetchJobDetails(jobId); // Llamar a la función para obtener los detalles del trabajo
    } else {
      throw new Error('No se pudo obtener el ID del trabajo.');
    }
  } catch (error) {
    console.error('Error:', error);
    alert(`Hubo un error al crear la orden. Por favor, intenta de nuevo. Error: ${error.message}`);
  }
}

// Función para obtener los detalles de un trabajo (orden) por su ID
async function fetchJobDetails(jobId) {
  const apiKey = 'yoJYongi4V4m0S4LClubdyiu5nq6VIpxazcFaghi';
  const url = `https://api.xandar.instaleap.io/jobs/${jobId}`;

  try {
    // Realizar una solicitud GET a la API para obtener los detalles del trabajo
    const response = await fetch(url, {
      method: 'GET',
      headers: {
        'Accept': 'application/json',
        'x-api-key': apiKey
      }
    });

    // Manejar errores si la respuesta no es exitosa
    if (!response.ok) {
      const errorText = await response.text();
      throw new Error(`Error al obtener detalles del trabajo: ${errorText}`);
    }

    // Obtener los detalles del trabajo como JSON
    const jobDetails = await response.json();
    console.log('Detalles del trabajo:', jobDetails);

    // Mostrar los detalles del trabajo en la interfaz de usuario
    displayJobDetails(jobDetails);
  } catch (error) {
    console.error('Error:', error);
    alert(`Hubo un error al obtener los detalles del trabajo. Error: ${error.message}`);
  }
}

// Mostrar los detalles del trabajo en la interfaz de usuario
function displayJobDetails(jobDetails) {
    const detailsContainer = document.createElement('div');
    detailsContainer.classList.add('job-details');
  
    // Proyección de los detalles importantes en el front
    detailsContainer.innerHTML = `
      <h2>Detalles de la orden</h2>
      <p><strong>ID de Cliente:</strong> ${jobDetails.client_id}</p>
      <p><strong>Origen:</strong> ${jobDetails.origin.name}, ${jobDetails.origin.address}, ${jobDetails.origin.city}, ${jobDetails.origin.country}</p>
      <p><strong>Destino:</strong> ${jobDetails.destination.name}, ${jobDetails.destination.address}</p>
      <p><strong>Monto:</strong> ${jobDetails.products_cost.amount} ${jobDetails.products_cost.currency}</p>
      <button id="boton_facturar">Facturar</button> <!-- Botón para facturar -->
    `;
  
    // Agregar los detalles al contenedor principal en la página
    const mainContainer = document.getElementById('detalles_de_pago');
    mainContainer.appendChild(detailsContainer);
  
    // Obtener el jobId de jobDetails
    const jobId = jobDetails.id;
  
    // Agregar evento click al botón "Facturar"
    const botonFacturar = document.getElementById('boton_facturar');
    botonFacturar.addEventListener('click', () => {
      facturarOrden(jobId, jobDetails.client_id);
    });
  }
  
  // Función para facturar una orden
  async function facturarOrden(jobId, clientId) {
    const apiKey = 'yoJYongi4V4m0S4LClubdyiu5nq6VIpxazcFaghi';
    const url = `https://api.xandar.instaleap.io/jobs/${jobId}/payment_info`;
  
    try {
      // Realizar una solicitud PUT a la API para facturar la orden
      const response = await fetch(url, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Accept': 'application/json',
          'x-api-key': apiKey
        },
        body: JSON.stringify({
          prices: {
            subtotal: parseFloat(document.getElementById('valor').value.trim()), // Subtotal toma el valor de pago ingresado
            shipping_fee: 0,
            discounts: 0,
            taxes: 0,
            order_value: parseFloat(document.getElementById('valor').value.trim()), // Order_value toma el valor de pago ingresado
            attributes: [
              {
                type: 'SUBTOTAL',
                name: document.getElementById('valor').value.trim(),
                value: parseFloat(document.getElementById('valor').value.trim()) // Atributo value toma el valor de pago ingresado
              }
            ]
          },
          payment: {
            payment_status: 'SUCCEEDED',
            method: 'CASH',
            id: clientId, // ID toma el client_id
            reference: '4059310181757001',
            value: parseFloat(document.getElementById('valor').value.trim()) // Value toma el valor de pago ingresado
          }
        })
      });
  
      // Manejar errores si la respuesta no es exitosa
      if (!response.ok) {
        const errorText = await response.text();
        throw new Error(`Error al facturar la orden: ${errorText}`);
      }
  
      // Obtener los datos de la respuesta como JSON
      const responseData = await response.json();
      console.log('Orden facturada:', responseData);
      alert('Orden facturada exitosamente.');
    } catch (error) {
      console.error('Error:', error);
      alert(`Hubo un error al facturar la orden. Error: ${error.message}`);
    }
  }
  