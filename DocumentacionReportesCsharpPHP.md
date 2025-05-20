#   Base de Datos.
#   Documentación del Proyecto: Reportes en C# y PHP.

##   19/5/25

##   Kianna Pascual.

##   5to. A. Inf.

<br>
<br>

<center>
<img src="imagenes/Hiki.png" width="200">
</center>

## Tabla de Contenidos

1. [Introducción](#introducción)
2. [Base de Datos](#basededatos)
3. [Reporte en C# - con ReportViewer y ADO.NET](#reporte-en-c-con-reportviewer-y-adonet)
    * [Instalación de NuGet Package](#instalación-de-nuget-package)

    * [Archivos C#](#archivos-c)
    * [Código C# (ReporteForm.cs)](#código-c-reporteformcs)
    * [Código C# (CRUDfacturas.cs)](#código-c-crudfacturascs)
    * [Reporte .rdlc](#reporte-rdlc)
4. [Reporte PHP - con Dompdf](#reporte-php-con-dompdf)
    * [Instalación de Composer](#instalación-de-composer)

    * [Instalación de Dompdf](#instalación-de-dompdf)
    * [Archivos PHP](#archivos-php)
    * [Código PHP (Formulario.html)](#código-php-formulariohtml)
    * [Código PHP (generar_reporte.php)](#código-php-generar_reportephp)
5. [Conclusión](#conclusión)

##   Introducción

Esta documentación describe la implementación y el paso a paso para hacer reportes en C# y PHP para una aplicación de facturación. Los reportes son una parte fundamental de las aplicaciones de software, ya que presentan información de manera organizada y fácil de entender. En este proyecto, se utilizan diferentes tecnologías para generar los reportes en cada lenguaje: C# y PHP.

## Basededatos

La base de datos utilizada en este proyecto es una base de datos SQL Server llamada `BD_FacturacionPruebas`. Contiene una tabla única llamada `Factura`, que almacena la información de las facturas.



###   Estructura de la Tabla `Factura`

La tabla `Factura` tiene la siguiente estructura:

* **ID (INT, Primary Key):** Identificador único de la factura.

* **DESCRIPCION (VARCHAR(255)):** Descripción del producto o servicio facturado.
* **CATEGORIA (VARCHAR(100)):** Categoría del producto o servicio.
* **CANTIDAD (INT):** Cantidad del producto o servicio.
* **PRECIO\_UNITARIO (DECIMAL(10, 2)):** Precio por unidad del producto o servicio.
* **ITEBIS (DECIMAL(10, 2)):** Impuesto sobre Transferencia de Bienes Industrializados y Servicios (ITBIS).
* **DESCUENTO (DECIMAL(10, 2)):** Descuento aplicado a la factura.
* **TOTAL\_GENERAL (DECIMAL(10, 2)):** Total general de la factura (calculado como (Cantidad \* Precio Unitario) + ITEBIS - Descuento).

```sql

--Creamos la base de datos
CREATE DATABASE BD_FacturacionPruebas;
GO

--Usando la base de datos ya creada, creamos sus campos
USE BD_FacturacionPruebas;

CREATE TABLE Factura (
    ID INT IDENTITY PRIMARY KEY,
    DESCRIPCION VARCHAR(250),
    CATEGORIA VARCHAR(50),
    CANTIDAD INT,
    PRECIO_UNITARIO DECIMAL(10),
    ITEBIS DECIMAL(10),
    DESCUENTO DECIMAL(10),
    TOTAL_GENERAL DECIMAL(10)
);
GO

--Usando la base datos, ahora insertamos algunos registros
USE BD_FacturacionPruebas;

INSERT INTO Factura (DESCRIPCION, CATEGORIA, CANTIDAD, PRECIO_UNITARIO, ITEBIS,
DESCUENTO, TOTAL_GENERAL)
VALUES
    ('Laptop', 'Electronica', 2, 750, 270, 75, 1695), 
    ('Impresora HP', 'Electronica', 1, 300, 54, 30, 324), 
    ('Papel Bond A4', 'Oficina', 5, 10, 9, 0, 59),       
    ('Escritorio de Oficina', 'Mobiliario', 1, 200, 36, 20, 216);

```

<br>

<center>
<img src="imagenes/BaseDatosTabla.jpg" width="700">
</center>

## Reporte en c# con reportviewer y adonet

En C#, los reportes se pueden generar de distintas formas, sin embargo, en este caso, lo crearemos utilizando ReportViewer y ADO.NET.

###  <u> Instalación de nuget package </u>

Se requiere la instalación del paquete NuGet `Microsoft.ReportingServices.ReportViewerControl.WinForms`. Esto se puede hacer desde el Administrador de paquetes NuGet en Visual Studio.

###  <u> Archivos C# </u>

Se crean los siguientes archivos:

* `ReporteForm.cs`: Formulario para mostrar el reporte.
* `CRUDfacturas.cs`: Formulario para el CRUD de facturas.
* `ReporteFactura.rdlc`: Archivo de diseño del reporte.

###  <u> Código c# reporteformcs </u>

Este formulario carga y muestra el reporte de facturas utilizando el control ReportViewer. Ten en cuenta que algunos factores cambian según cada usuario.

####   **Por ejemplo:**

La parte de la conexión a la base de datos va a variar dependiendo de tu base de datos, el nombre del servidor, del usuario o si tiene o no "**Windows Authentication**" en el caso de ser **SQL Server** que es donde se encuentra esta base de datos.

<br>

<center>
<img src="imagenes/ReporteFacturaForm.jpg" width="700">
</center>

```csharp
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using Microsoft.Reporting.WinForms;
using System.Data.SqlClient;

namespace Facturas
{
    public partial class ReporteForm : Form
    {
        private string connectionString = "Data Source=ENZOACER\\SQLEXPRESS;Initial Catalog=BD_FacturacionPruebas;Integrated Security=True;";

        // Puede variar la conexión a la base de datos

        public ReporteForm()
        {
            InitializeComponent();
        }

        private void ReporteForm_Load(object sender, EventArgs e)
        {
            CargarReporte();
        }

        private void CargarReporte()
        {
            // 1. Establecer la ruta del reporte .rdlc
            reportViewer1.LocalReport.ReportEmbeddedResource = "Facturas.ReporteFactura.rdlc";

            // 2. Obtener los datos de las facturas desde la base de datos
            List<Factura> listaFacturas = ObtenerFacturasDesdeBD();

            // 3. Crear un origen de datos para el reporte
            ReportDataSource rds = new ReportDataSource("DataSet1", listaFacturas);

            // 4. Limpiar los orígenes de datos existentes y agregar el nuevo
            reportViewer1.LocalReport.DataSources.Clear();
            reportViewer1.LocalReport.DataSources.Add(rds);

            // 5. Actualizar el reporte para mostrar los datos
            reportViewer1.RefreshReport();
        }

        private List<Factura> ObtenerFacturasDesdeBD()
        {
            List<Factura> facturas = new List<Factura>();

            // 1. Crear una conexión a la base de datos SQL Server
            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                try
                {
                    // 2. Abrir la conexión
                    connection.Open();

                    // 3. Definir la consulta SQL para seleccionar los datos de las facturas
                    string query = "SELECT ID, DESCRIPCION, CATEGORIA, CANTIDAD, PRECIO_UNITARIO, ITEBIS, DESCUENTO, TOTAL_GENERAL FROM Factura";

                    // 4. Crear un comando SQL con la consulta y la conexión
                    using (SqlCommand command = new SqlCommand(query, connection))
                    {
                        // 5. Ejecutar la consulta y obtener un lector de datos
                        using (SqlDataReader reader = command.ExecuteReader())
                        {
                            // 6. Leer los datos fila por fila
                            while (reader.Read())
                            {
                                // 7. Crear un objeto Factura y asignar los valores de cada columna
                                Factura factura = new Factura
                                {
                                    ID = reader.GetInt32(reader.GetOrdinal("ID")),
                                    DESCRIPCION = reader.GetString(reader.GetOrdinal("DESCRIPCION")),
                                    CATEGORIA = reader.GetString(reader.GetOrdinal("CATEGORIA")),
                                    CANTIDAD = reader.GetInt32(reader.GetOrdinal("CANTIDAD")),
                                    PRECIO_UNITARIO = reader.GetDecimal(reader.GetOrdinal("PRECIO_UNITARIO")),
                                    ITEBIS = reader.GetDecimal(reader.GetOrdinal("ITEBIS")),
                                    DESCUENTO = reader.GetDecimal(reader.GetOrdinal("DESCUENTO")),
                                    TOTAL_GENERAL = reader.GetDecimal(reader.GetOrdinal("TOTAL_GENERAL"))
                                };
                                // 8. Agregar la factura a la lista
                                facturas.Add(factura);
                            }
                        }
                    }
                }
                catch (SqlException ex)
                {
                    // 9. Manejar excepciones de SQL (errores de base de datos)
                    MessageBox.Show($"Error al cargar los datos: {ex.Message}");
                    return null;
                }
            }
            // 10. Devolver la lista de facturas
            return facturas;
        }

        private void CargarBTN_Click(object sender, EventArgs e)
        {
            // 1. Volver a cargar el reporte con los datos actualizados
            CargarReporte();
            // 2. Refrescar la vista del reporte
            this.reportViewer1.RefreshReport();
        }

        private void CRUDbtn_Click(object sender, EventArgs e)
        {
            // 1. Ocultar el formulario actual
            this.Hide();
            // 2. Crear una instancia del formulario CRUDfacturas
            CRUDfacturas CRUDfacturas = new CRUDfacturas();
            // 3. Mostrar el formulario CRUDfacturas
            CRUDfacturas.Show();
        }
    }

    public class Factura
    {
        public int ID { get; set; }
        public string DESCRIPCION { get; set; }
        public string CATEGORIA { get; set; }
        public int CANTIDAD { get; set; }
        public decimal PRECIO_UNITARIO { get; set; }
        public decimal ITEBIS { get; set; }
        public decimal DESCUENTO { get; set; }
        public decimal TOTAL_GENERAL { get; set; }
    }
}

```

###   <u> Código c# crudfacturascs </u>

   Este formulario permite realizar operaciones CRUD (Crear, Leer, Actualizar, Eliminar) sobre las facturas en la base de datos.

   <br>

   <center>
<img src="imagenes/CrudFacturasForm.jpg" width="700">
</center>

   ```csharp
   
   using System;
   using System.Collections.Generic;
   using System.ComponentModel;
   using System.Data;
   using System.Drawing;
   using System.Linq;
   using System.Text;
   using System.Threading.Tasks;
   using System.Windows.Forms;
   using System.Data.SqlClient;
   using System.Globalization;
   using System.Diagnostics;

   namespace Facturas
   {
       public partial class CRUDfacturas : Form
       {
           //  Cadena de conexión a la base de datos SQL Server
           private string connectionString = "Data Source=ENZOACER\\SQLEXPRESS;Initial Catalog=BD_FacturacionPruebas;Integrated Security=True;";

           //  BindingSource para enlazar los datos del DataGridView
           private BindingSource facturasBindingSource = new BindingSource();

           //  Variables para almacenar temporalmente los valores de las celdas
           private object cantidad;
           private object precioUnitario;
           private object itebis;
           private object descuento;
           private object totalIngresado;

           //  Constructor de la clase
           public CRUDfacturas()
           {
               InitializeComponent();  // Inicializa los componentes visuales del formulario
               ConfigurarDataGridView(); // Configura las columnas del DataGridView
               CargarFacturas();  // Carga los datos de las facturas desde la base de datos
           }

           //  Configura las columnas del DataGridView
           private void ConfigurarDataGridView()
           {
               dataGridView1.AutoGenerateColumns = false;  // Desactiva la generación automática de columnas

               //  Añade las columnas al DataGridView
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "ID", DataPropertyName = "ID", Name = "ID" });
               dataGridView1.Columns["ID"].ReadOnly = true;  // La columna ID es de solo lectura
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Descripción", DataPropertyName = "DESCRIPCION", Name = "Descripcion" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Categoría", DataPropertyName = "CATEGORIA", Name = "Categoria" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Cantidad", DataPropertyName = "CANTIDAD", Name = "Cantidad" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Precio Unitario", DataPropertyName = "PRECIO_UNITARIO", Name = "PrecioUnitario" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "ITEBIS", DataPropertyName = "ITEBIS", Name = "ITEBIS" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Descuento", DataPropertyName = "DESCUENTO", Name = "Descuento" });
               dataGridView1.Columns.Add(new DataGridViewTextBoxColumn { HeaderText = "Total General", DataPropertyName = "TOTAL_GENERAL", Name = "TotalGeneral" });
               dataGridView1.Columns["TotalGeneral"].ReadOnly = false;  // La columna TotalGeneral es editable

               dataGridView1.AllowUserToAddRows = true;  
               // Permite al usuario añadir nuevas filas

               dataGridView1.UserAddedRow += DataGridView1_UserAddedRow;  
               // Asocia un evento cuando se añade una fila

               dataGridView1.CellEndEdit += DataGridView1_CellEndEdit;  
               // Asocia un evento cuando termina la edición de una celda

               dataGridView1.DataError += DataGridView1_DataError;  
               // Asocia un evento para manejar errores de datos
           }

           //  Carga los datos de las facturas desde la base de datos
           private void CargarFacturas()
           {
               try
               {
                   using (SqlConnection connection = new SqlConnection(connectionString))
                   {
                       connection.Open();
                       string query = "SELECT ID, DESCRIPCION, CATEGORIA, CANTIDAD, PRECIO_UNITARIO, ITEBIS, DESCUENTO, TOTAL_GENERAL FROM Factura";
                       using (SqlDataAdapter adapter = new SqlDataAdapter(query, connection))
                       {
                           DataTable dt = new DataTable();
                           adapter.Fill(dt);
                           facturasBindingSource.DataSource = dt;
                           dataGridView1.DataSource = facturasBindingSource;
                       }
                   }
               }
               catch (SqlException ex)
               {
                   MessageBox.Show($"Error al cargar las facturas: {ex.Message}");
               }
           }

           //  Evento que se ejecuta cuando el usuario añade una nueva fila al DataGridView
        private void DataGridView1_UserAddedRow(object sender, DataGridViewRowEventArgs e)
        {
            CrearBTN.Enabled = true;     // Habilita el botón Crear
            ActualizarBTN.Enabled = false; // Deshabilita el botón Actualizar
            EliminarBTN.Enabled = false;  // Deshabilita el botón Eliminar
        }

        //  Evento que se ejecuta cuando el usuario termina de editar una celda del DataGridView
        private void DataGridView1_CellEndEdit(object sender, DataGridViewCellEventArgs e)
        {
            //  Verifica si la columna editada es Cantidad, Precio Unitario, ITEBIS o Descuento
            if (e.ColumnIndex == dataGridView1.Columns["Cantidad"].Index ||
                e.ColumnIndex == dataGridView1.Columns["PrecioUnitario"].Index ||
                e.ColumnIndex == dataGridView1.Columns["ITEBIS"].Index ||
                e.ColumnIndex == dataGridView1.Columns["Descuento"].Index)
            {
                DataGridViewRow currentRow = dataGridView1.Rows[e.RowIndex]; // Obtiene la fila actual

                //  Intenta obtener y convertir los valores de las celdas a los tipos de datos correctos
                if (currentRow.Cells["Cantidad"].Value != null &&
                    decimal.TryParse(currentRow.Cells["Cantidad"].Value.ToString(), out decimal cantidad) &&
                    currentRow.Cells["PrecioUnitario"].Value != null &&
                    decimal.TryParse(currentRow.Cells["PrecioUnitario"].Value.ToString(),
                                   NumberStyles.Any, CultureInfo.InvariantCulture, out decimal precioUnitario) &&
                    currentRow.Cells["ITEBIS"].Value != null &&
                    decimal.TryParse(currentRow.Cells["ITEBIS"].Value.ToString(),
                                   NumberStyles.Any, CultureInfo.InvariantCulture, out decimal itebis) &&
                    currentRow.Cells["Descuento"].Value != null &&
                    decimal.TryParse(currentRow.Cells["Descuento"].Value.ToString(),
                                   NumberStyles.Any, CultureInfo.InvariantCulture, out decimal descuento))
                {
                    //  Calcula el Total General
                    decimal totalGeneral = (cantidad * precioUnitario) + itebis - descuento;
                    currentRow.Cells["TotalGeneral"].Value = totalGeneral; // Actualiza la celda Total General
                }
                else
                {
                    currentRow.Cells["TotalGeneral"].Value = 0;  // Si hay errores en la conversión, el Total General es 0
                }
            }
        }

        private void CrearBTN_Click_1(object sender, EventArgs e)
        {
            //  Itera a través de cada fila del DataGridView
            foreach (DataGridViewRow row in dataGridView1.Rows)
            {
                //  Verifica que la fila no sea una fila nueva y que los campos Descripción y Categoría no estén vacíos
                if (!row.IsNewRow && row.Cells["Descripcion"].Value != null &&
                    !string.IsNullOrWhiteSpace(row.Cells["Descripcion"].Value.ToString()))
                {
                    //  Verifica que los campos Categoría, Cantidad, Precio Unitario, ITEBIS, Descuento y Total General tengan valores válidos
                    if (row.Cells["Categoria"].Value != null &&
                        !string.IsNullOrWhiteSpace(row.Cells["Categoria"].Value.ToString()) &&
                        row.Cells["Cantidad"].Value != null &&
                        int.TryParse(row.Cells["Cantidad"].Value.ToString(), out int cantidad) &&
                        row.Cells["PrecioUnitario"].Value != null &&
                        decimal.TryParse(row.Cells["PrecioUnitario"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal precioUnitario) &&
                        row.Cells["ITEBIS"].Value != null &&
                        decimal.TryParse(row.Cells["ITEBIS"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal itebis) &&
                        row.Cells["Descuento"].Value != null &&
                        decimal.TryParse(row.Cells["Descuento"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal descuento) &&
                        row.Cells["TotalGeneral"].Value != null &&
                        decimal.TryParse(row.Cells["TotalGeneral"].Value.ToString(), NumberStyles.Number,
                                       CultureInfo.InvariantCulture, out decimal totalGeneralCalculado))
                    {
                        try
                        {
                            //  Crea y abre una conexión a la base de datos SQL Server
                            using (SqlConnection connection = new SqlConnection(connectionString))
                            {
                                connection.Open();
                                //  Define la consulta SQL para insertar una nueva factura
                                string query = "INSERT INTO Factura (DESCRIPCION, CATEGORIA, " +
                                               "CANTIDAD, PRECIO_UNITARIO, ITEBIS, DESCUENTO, TOTAL_GENERAL) " +
                                               "VALUES (@Descripcion, @Categoria, @Cantidad, " +
                                               "@PrecioUnitario, @Itebis, @Descuento, @TotalGeneral)";
                                //  Crea un SqlCommand para ejecutar la consulta
                                using (SqlCommand command = new SqlCommand(query, connection))
                                {
                                    //  Añade los parámetros a la consulta con los valores de las celdas
                                    command.Parameters.AddWithValue("@Descripcion",
                                                                    row.Cells["Descripcion"].Value?.ToString() ?? "");
                                    command.Parameters.AddWithValue("@Categoria",
                                                                    row.Cells["Categoria"].Value?.ToString() ?? "");
                                    command.Parameters.AddWithValue("@Cantidad", cantidad);
                                    command.Parameters.AddWithValue("@PrecioUnitario", precioUnitario);
                                    command.Parameters.AddWithValue("@Itebis", itebis);
                                    command.Parameters.AddWithValue("@Descuento", descuento);
                                    command.Parameters.AddWithValue("@TotalGeneral",
                                                                    totalGeneralCalculado);
                                    //  Ejecuta la consulta y obtiene el número de filas afectadas
                                    int rowsAffected = command.ExecuteNonQuery();
                                    //  Si se insertó una fila correctamente
                                    if (rowsAffected > 0)
                                    {
                                        MessageBox.Show("Factura creada exitosamente.");
                                        CargarFacturas();    // Recarga los datos en el DataGridView
                                        CrearBTN.Enabled = false;
                                        ActualizarBTN.Enabled = false;
                                        EliminarBTN.Enabled = false;
                                        return;
                                    }
                                    else
                                    {
                                        MessageBox.Show("No se pudo crear la factura.");
                                        return;
                                    }
                                }
                            }
                        }
                        catch (SqlException ex)
                        {
                            MessageBox.Show($"Error al crear la factura (SQL): {ex.Message}");
                            return;
                        }
                        catch (Exception ex)
                        {
                            MessageBox.Show($"Error inesperado al crear la factura: {ex.Message}");
                            return;
                        }
                    }

                    else
                    {
                        MessageBox.Show("Por favor, complete todos los campos de la nueva factura \ncon valores válidos.", "Advertencia", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                        return;
                    }
                }
            }
            MessageBox.Show("Por favor, ingrese los detalles de la factura en una nueva fila antes \nde crear.", "Advertencia", MessageBoxButtons.OK, MessageBoxIcon.Warning);
        }

        private void ActualizarBTN_Click_1(object sender, EventArgs e)
        {
            //  Verifica si hay alguna fila seleccionada en el DataGridView
            if (dataGridView1.SelectedRows.Count > 0)
            {
                //  Obtiene la primera fila seleccionada
                DataGridViewRow selectedRow = dataGridView1.SelectedRows[0];
                //  Verifica si la fila seleccionada tiene un valor en la columna ID
                if (selectedRow.Cells["ID"].Value != null)
                {
                    //  Verifica que los campos Descripción, Categoría, Cantidad, Precio Unitario, ITEBIS y Descuento no estén vacíos y tengan valores válidos
                    if (selectedRow.Cells["Descripcion"].Value == null ||
                        string.IsNullOrWhiteSpace(selectedRow.Cells["Descripcion"].Value.ToString()) ||
                        selectedRow.Cells["Categoria"].Value == null ||
                        string.IsNullOrWhiteSpace(selectedRow.Cells["Categoria"].Value.ToString()) ||
                        selectedRow.Cells["Cantidad"].Value == null ||
                        !int.TryParse(selectedRow.Cells["Cantidad"].Value.ToString(), out int cantidad) ||
                        selectedRow.Cells["PrecioUnitario"].Value == null ||
                        !decimal.TryParse(selectedRow.Cells["PrecioUnitario"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal precioUnitario) ||
                        selectedRow.Cells["ITEBIS"].Value == null ||
                        !decimal.TryParse(selectedRow.Cells["ITEBIS"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal itebis) ||
                        selectedRow.Cells["Descuento"].Value == null ||
                        !decimal.TryParse(selectedRow.Cells["Descuento"].Value.ToString(), NumberStyles.Any,
                                       CultureInfo.InvariantCulture, out decimal descuento))
                    {
                        MessageBox.Show("Por favor, complete todos los campos.", "Error",
                                        MessageBoxButtons.OK, MessageBoxIcon.Error);
                        return;
                    }
                    else
                    {
                        //  Calcula el Total General esperado basado en los valores ingresados
                        decimal totalEsperado = (precioUnitario * cantidad) + itebis - descuento;
                        decimal totalIngresado = totalEsperado;

                        //  Verifica si se ingresó un valor en la columna Total General y si es un decimal válido
                        if (selectedRow.Cells["TotalGeneral"].Value != null &&
                            decimal.TryParse(selectedRow.Cells["TotalGeneral"].Value.ToString(),
                                           NumberStyles.Number, CultureInfo.InvariantCulture, out decimal totalUsuarioIngresado))
                        {
                            totalIngresado = totalUsuarioIngresado; // Usa el valor ingresado por el usuario
                        }
                        else
                        {
                            MessageBox.Show($"No se ingresó un Total General válido. El total correcto \nes: {totalEsperado:N2}", "Advertencia", MessageBoxButtons.OK, MessageBoxIcon.Warning);
                            selectedRow.Cells["TotalGeneral"].Value = totalEsperado; // Establece el valor correcto en la celda
                        }

                        //  Compara el Total General ingresado con el Total General esperado
                        if (totalIngresado != totalEsperado)
                        {
                            MessageBox.Show($"El Total General ingresado es incorrecto. El total \ncorrecto debería ser: {totalEsperado:N2}", "Error de Total", MessageBoxButtons.OK,
                                            MessageBoxIcon.Warning);
                            return;
                        }
                        //  Imprime en la ventana de depuración los valores antes de la actualización
                        Debug.WriteLine("ActualizarBTN_Click: Valores antes de la actualización:");
                        Debug.WriteLine($"  ID: {selectedRow.Cells["ID"].Value}");
                        Debug.WriteLine($"  Descripción: {selectedRow.Cells["Descripcion"].Value}");
                        Debug.WriteLine($"  Categoría: {selectedRow.Cells["Categoria"].Value}");
                        Debug.WriteLine($"  Cantidad: {cantidad}");
                        Debug.WriteLine($"  Precio Unitario: {precioUnitario}");
                        Debug.WriteLine($"  ITEBIS: {itebis}");
                        Debug.WriteLine($"  Descuento: {descuento}");
                        Debug.WriteLine($"  Total General Ingresado: {totalIngresado}");
                        Debug.WriteLine($"  Total General Esperado: {totalEsperado}");

                        try
                        {
                            using (SqlConnection connection = new SqlConnection(connectionString))
                            {
                                connection.Open();
                                //  Define la consulta SQL para actualizar la factura
                                string query = "UPDATE Factura SET DESCRIPCION = @Descripcion, CATEGORIA = @Categoria, CANTIDAD = @Cantidad, " +
                                               "PRECIO_UNITARIO = @PrecioUnitario, ITEBIS = @Itebis, DESCUENTO = @Descuento, TOTAL_GENERAL = @TotalGeneral " +
                                               "WHERE ID = @ID";
                                //  Crea un SqlCommand para ejecutar la consulta
                                using (SqlCommand command = new SqlCommand(query, connection))
                                {
                                    //  Añade los parámetros a la consulta con los valores de las celdas
                                    command.Parameters.AddWithValue("@ID", Convert.ToInt32(selectedRow.Cells["ID"].Value));
                                    command.Parameters.AddWithValue("@Descripcion", selectedRow.Cells["Descripcion"].Value?.ToString() ?? "");
                                    command.Parameters.AddWithValue("@Categoria", selectedRow.Cells["Categoria"].Value?.ToString() ?? "");
                                    command.Parameters.AddWithValue("@Cantidad", cantidad);
                                    command.Parameters.AddWithValue("@PrecioUnitario", precioUnitario);
                                    command.Parameters.AddWithValue("@Itebis", itebis);
                                    command.Parameters.AddWithValue("@Descuento", descuento);
                                    command.Parameters.AddWithValue("@TotalGeneral", totalIngresado);

                                    //  Ejecuta la consulta y obtiene el número de filas afectadas
                                    int rowsAffected = command.ExecuteNonQuery();
                                    //  Si se actualizó la fila correctamente
                                    if (rowsAffected > 0)
                                    {
                                        MessageBox.Show("Factura actualizada exitosamente.");
                                        CargarFacturas();    // Recarga los datos en el DataGridView
                                    }
                                    else
                                    {
                                        MessageBox.Show("No se pudo actualizar la factura.");
                                    }
                                }
                            }
                        }
                        catch (SqlException ex)
                        {
                            MessageBox.Show($"Error al actualizar la factura: {ex.Message}");
                        }
                    }
                }
                else
                {
                    MessageBox.Show("Por favor, seleccione una fila válida para actualizar.");
                }
            }
            else
            {
                MessageBox.Show("Por favor, seleccione una fila para actualizar.");
            }
        }

        private void DebugWriteLine(string v)
        {
            throw new NotImplementedException();
        }

        private void EliminarBTN_Click_1(object sender, EventArgs e)
        {
            //  Verifica si hay alguna fila seleccionada en el DataGridView
            if (dataGridView1.SelectedRows.Count > 0)
            {
                //  Obtiene la primera fila seleccionada
                DataGridViewRow selectedRow = dataGridView1.SelectedRows[0];

                //  Verifica si la fila seleccionada tiene un valor en la columna ID
                if (selectedRow.Cells["ID"].Value != null)
                {
                    //  Muestra un cuadro de diálogo de confirmación antes de eliminar la factura
                    if (MessageBox.Show("¿Está seguro que desea eliminar esta factura?", "Confirmar Eliminación", MessageBoxButtons.YesNo, MessageBoxIcon.Warning) == DialogResult.Yes)
                    {
                        try
                        {
                            //  Crea y abre una conexión a la base de datos SQL Server
                            using (SqlConnection connection = new SqlConnection(connectionString))
                            {
                                connection.Open();
                                //  Define la consulta SQL para eliminar la factura por su ID
                                string query = "DELETE FROM Factura WHERE ID = @ID";
                                //  Crea un SqlCommand para ejecutar la consulta
                                using (SqlCommand command = new SqlCommand(query, connection))
                                {
                                    //  Añade el parámetro ID a la consulta
                                    command.Parameters.AddWithValue("@ID", Convert.ToInt32(selectedRow.Cells["ID"].Value));

                                    //  Ejecuta la consulta y obtiene el número de filas afectadas
                                    int rowsAffected = command.ExecuteNonQuery();
                                    //  Si se eliminó la fila correctamente:
                                    if (rowsAffected > 0)
                                    {
                                        MessageBox.Show("Factura eliminada exitosamente.");
                                        CargarFacturas();    // Recarga los datos en el DataGridView
                                    }
                                    else
                                    {
                                        MessageBox.Show("No se pudo eliminar la factura.");
                                    }
                                }
                            }
                        }
                        catch (SqlException ex)
                        {
                            MessageBox.Show($"Error al eliminar la factura: {ex.Message}");
                        }
                    }
                }
                else
                {
                    MessageBox.Show("Por favor, seleccione una fila válida para eliminar.");
                }
            }
            else
            {
                MessageBox.Show("Por favor, seleccione una fila para eliminar.");
            }
        }

        private void VolverReporteBTN_Click_1(object sender, EventArgs e)
        {
            ReporteForm reporteForm = new ReporteForm();
            reporteForm.Show();   // Muestra el formulario ReporteForm
            this.Hide();       // Oculta el formulario actual
        }

        //  Evento que se ejecuta cuando ocurre un error de datos en el DataGridView
        private void DataGridView1_DataError(object sender, DataGridViewDataErrorEventArgs e)
        {
            //  Maneja los errores de formato 
            if (e.Exception is FormatException)
            {
                MessageBox.Show("Por favor, ingrese un formato numérico válido sin separadores de miles (ej: 1000 en lugar de 1,000).", "Error de Formato", MessageBoxButtons.OK, MessageBoxIcon.Error);
                e.ThrowException = false;  // Indica que el error ha sido manejado y no debe lanzarse una excepción
            }
        }
    }
}

```

###  <u> Reporte rdlc </u>

<center>
<img src="imagenes/ReporteFacturasRdlc.jpg" width="700">
</center>
<br>

Este archivo es el diseño del reporte en C#.  Aquí se define la estructura visual del reporte, incluyendo los campos que se mostrarán (como ID, Descripción, Categoría, etc.) y su formato.  También se configura la conexión a la base de datos y la tabla de la cual se obtendrán los datos.

Luego en "**ReporteForm.cs**" (como ya vimos previamente en la parte de arriba) en el botón de cargar reporte, se especifica la ruta del archivo "**ReporteFactura.rdlc**" para que se muestre en el "**ReportViewer**".

Ahora vamos a ver como se hace un reporte de PHP.

## Reporte php con dompdf

Para generar reportes en PHP, en este caso, se utilizó la librería Dompdf.

###  <u> Instalación de composer </u>

Primero, se debe instalar Composer, que es un gestor de dependencias para PHP.

<br>

<center>
<img src="imagenes/DescargarComposer.jpg" width="700">
</center>

<br>

1.  Descargar y ejecutar el instalador "Composer-Setup.exe" desde la página oficial. ( [ Click aquí para ir a la página oficial para instalar Composer ](#https://getcomposer.org/) )

2.  Seguir las instrucciones del instalador, asegurándose de seleccionar la ruta correcta al archivo `php.exe` de XAMPP.

3.  Configurar el proxy si es necesario (en este caso no) y finalizar la instalación. 

###  <u> Instalación de Dompdf: </u>

Una vez instalado Composer, se instala la librería Dompdf en la carpeta del proyecto.

<br>

<center>
<img src="imagenes/CmdComposer.jpg" width="700">
</center>

<br>

1.  Abrir la línea de comandos y navegar hasta la carpeta del proyecto (donde se encuentra el archivo `composer.json`).
2.  Ejecutar el siguiente comando:

    ```bash
    composer require dompdf/dompdf
    ```

    Esto instalará Dompdf y sus dependencias en la carpeta `vendor` del proyecto. [cite: 191]

###  <u> Archivos PHP: </u>

<br>

<img src="imagenes/CarpetaReportePHP.jpg" width="200">
</center>

<br>

La estructura de este proyecto PHP en particular incluye los siguientes archivos y carpetas:

* `vendor/`:  Contiene Composer, Dompdf y sus dependencias.

* `autoload.php`:  Archivo generado por Composer para cargar automáticamente las clases.
* `composer.json` y `composer.lock`: Archivos de configuración de Composer.
* `Formulario.html`:  Formulario HTML para iniciar la generación del reporte.
* `generar_reporte.php`:  Script PHP que genera el reporte PDF.
* `Hiki cabecita con lengua.png`:  Imagen utilizada en el formulario (de mi ranita).


### <u> Código PHP. Formulario.html: </u>

Este archivo HTML contiene el formulario con un botón que al ser presionado, llama al script `generar_reporte.php` para crear el reporte.

<br>

<img src="imagenes/FormularioHtml.jpg" width="700">
</center>


<br>

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Generar reporte de facturas de la base de datos BD_FacturacionPrueba</title>
</head>
<body bgcolor="#b034cf" style="margin: 30px;">
    <hr size="5" color="white" width="1350" align="center">
    <hr size="10" color="white" width="1350" align="center">
    <table border="0" width="1360" height="60" cellpadding="2" bgcolor="white">
        <tr>
            <th rowspan="1"> <font size="100" face="Agency FB"> ~ Generar Reporte de Factura ~ <br> (｡･∀･)ﾉﾞ </font></th>
            <th rowspan="1"> <img src="Hiki cabecita con lengua.png" height="350"/> </th> 
        </tr>
    </table>
    <hr size="5" color="white" width="1350" align="center">
    <hr size="10" color="white" width="1350" align="center">
    <br>
    <center>
        <form action="generar_reporte.php" method="post">
            <button type="submit" style="padding: 15px 30px; font-family: Agency FB; font-size: 30px;">Generar Reporte PDF</button>
        </form>
    </center>
</body>
</html>

```

### <u> Código PHP. generar_reporte.php: </u>

Este script PHP es el encargado de generar el reporte PDF utilizando la librería Dompdf.

1.  Incluye el autoloader de Composer para cargar las clases de Dompdf.

2.  Establece la conexión a la base de datos SQL Server.
3.  Ejecuta una consulta SQL para obtener los datos de la tabla `Factura`.
4.  Construye el código HTML para el reporte, incluyendo una tabla con los datos.
5.  Crea una instancia de la clase `Dompdf`.
6.  Carga el HTML en Dompdf.
7.  Genera el PDF a partir del HTML.
8.  Envía el PDF al navegador para su descarga.
9.  Cierra la conexión a la base de datos.

<br>

<img src="imagenes/PdfReporteFactura.jpg" width="700">
</center>


<br>

```php
<?php
// Incluir el autoloader de Composer
require 'vendor/autoload.php';
use Dompdf\Dompdf;

$serverName = "ENZOACER\SQLEXPRESS"; 
$database = "BD_FacturacionPruebas";
$connectionInfo = array( "Database" => $database, "CharacterSet" => "UTF-8" );
$conn = sqlsrv_connect( $serverName, $connectionInfo);

if( $conn === false ) {
    echo "Error al conectar a la base de datos.<br />";
    die( print_r( sqlsrv_errors(), true));
}

$sql = "SELECT ID, DESCRIPCION, CATEGORIA, CANTIDAD, PRECIO_UNITARIO, ITEBIS, DESCUENTO, TOTAL_GENERAL FROM Factura";
$stmt = sqlsrv_query( $conn, $sql );

if( $stmt === false ) {
    echo "Error al ejecutar la consulta.<br />";
    die( print_r( sqlsrv_errors(), true));
}

// Crear el contenido HTML para el PDF
$html = '
<!DOCTYPE html>
<html>
<head>
    <title>Reporte de Factura</title>
    <style>
        table {
            width: 100%;
            border-collapse: collapse;
        }
        th, td {
            border: 1px solid black;
            padding: 8px;
            text-align: left;
        }
        th {
            background-color: #f2f2f2;
        }
    </style>
</head>
<body>
    <h1>Reporte de Factura</h1>
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Descripción</th>
                <th>Categoría</th>
                <th>Cantidad</th>
                <th>Precio Unitario</th>
                <th>ITEBIS</th>
                <th>Descuento</th>
                <th>Total General</th>
            </tr>
        </thead>
        <tbody>';

while( $row = sqlsrv_fetch_array( $stmt, SQLSRV_FETCH_ASSOC) ) {
    $html .= '<tr>';
    $html .= '<td>' . $row['ID'] . '</td>';
    $html .= '<td>' . htmlspecialchars($row['DESCRIPCION'], ENT_QUOTES, 'UTF-8') . '</td>';
    $html .= '<td>' . htmlspecialchars($row['CATEGORIA'], ENT_QUOTES, 'UTF-8') . '</td>';
    $html .= '<td>' . $row['CANTIDAD'] . '</td>';
    <span class="math-inline">html \.\= '<td\></span>' . number_format($row['PRECIO_UNITARIO'], 2) . '</td>';
    <span class="math-inline">html \.\= '<td\></span>' . number_format($row['ITEBIS'], 2) . '</td>';
    <span class="math-inline">html \.\= '<td\></span>' . number_format($row['DESCUENTO'], 2) . '</td>';
    <span class="math-inline">html \.\= '<td\></span>' . number_format($row['TOTAL_GENERAL'], 2) . '</td>';
    $html .= '</tr>';
}

$html .= '
        </tbody>
    </table>
</body>
</html>
';

// Crear una instancia de Dompdf
$dompdf = new Dompdf();

// Cargar el HTML al Dompdf
$dompdf->loadHtml($html, 'UTF-8');

// Renderizar el HTML como PDF
$dompdf->render();

// Enviar el PDF al navegador para descargar
$dompdf->stream("reporte_factura.pdf", array("Attachment" => 0));

// Cerrar la conexion
sqlsrv_free_stmt( $stmt);
sqlsrv_close( $conn);
?>

```

### <u> ¡Atención! Nota importante </u>
 La funcionalidad de generación de reportes en PHP depende de un entorno de servidor web. En este caso,  **XAMPP debe estar en ejecución** para que el código PHP pueda procesar las solicitudes y acceder a la base de datos.  Verifica que los servicios de Apache y la base de datos estén iniciados en el panel de control de XAMPP.

<br>

## Conclusión
Este documento describe la implementación de la generación de reportes en C# y PHP, detallando el código y la configuración necesarios para cada lenguaje. Es importante conocer como hacer reportes en distintos entornos, ya qye estos nos ayudan a sacar información rapidamente de una base de datos.