#Canducci DataTables Entity Framework - NET 4.5

###(version: 1.0.0)

[![Canducci DataTables Entity Framework - NET 4.5](http://i666.photobucket.com/albums/vv25/netdragoon/1445749817_f-table-cell_128_zpschpxbzxv.png)](https://www.nuget.org/packages/Canducci.DataTables.EntityFramework/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package Canducci.DataTables.EntityFramework

```

##Como utilizar?

__Controller__

Declare os `namespace`:

```Csharp
using Canducci.DataTables;
using Canducci.DataTables.Contracts;

```

Utilizando o EntityFramework faça em seu `controller`:

1) Um método para mostrar uma `View`:

```Csharp

[HttpGet()]
[Route("peoples")]
public ActionResult Peoples()
{

    return View();

}

```

2) Um outro método que é responsável na geração do `Json`:

```Csharp
[HttpPost()]
public JsonResult PeoplesToResult(DataTablesConfig config)
{

    IDataTablesResult _result = null;

    using (myDataBaseEntities dbase = new myDataBaseEntities())
    {
        
        var query = dbase
            .Peoples
            .AsNoTracking()
            .Select(x => new { x.Id, x.Name })
            .AsQueryable(); 

        if (config.Filter.Name.Equals("Name") && !string.IsNullOrEmpty(config.Filter.Value))
        {
            _result = query.ToDataTables(config, x => x.Name.Contains(config.Filter.Value));
        }
        else if (config.Filter.Name.Equals("Id") && !string.IsNullOrEmpty(config.Filter.Value))
        {
            int Id;
            if (int.TryParse(config.Filter.Value, out Id))
                _result = query.ToDataTables(config, x => x.Id == Id);
            else
                _result = query.ToDataTables(config);
        }
        else
            _result = query.ToDataTables(config);

    }

    return Json(_result, JsonRequestBehavior.DenyGet);

}

```

A classe `DataTablesConfig` `config` é responsável em resgatar os dados enviados via post pelo
`jquery.datatables`. Além disso o método extensivo `ToDataTables` tem a obrigação de pegar essas
configurações e gerar o resultado esperado pelo `jquery.datatables`, inclusive ele gera automáticamente ordernação, então, se preocupe em criar os dados que serão enviados para tela, e o filtro (`Where`), o `OrderBy` é feito automáticamente pelo `ToDataTables`.

__View__

```Csharp

@{
    Layout = null;
}
<!DOCTYPE html>
<html>
<head>
    <meta name="viewport" content="width=device-width" />
    <title>Index</title>
    <link href="~/Content/bootstrap.min.css" rel="stylesheet" />
    <link href="~/Content/DataTables/css/dataTables.bootstrap.min.css" rel="stylesheet" />
    <script src="~/Scripts/jquery-1.10.2.min.js"></script>
    <script src="~/Scripts/bootstrap.min.js"></script>
    <script src="~/Scripts/DataTables/jquery.dataTables.min.js"></script>
    <script src="~/Scripts/DataTables/dataTables.bootstrap.js"></script>
</head>
<body>
    <div class="container">
        <br />
        <div class="form-inline">
            <div class="form-group">
                <select id="SelectFiltro" class="form-control select-info">
                    <option value="Id">Pelo Código</option>
                    <option value="Name">Pelo Nome</option>
                </select>
            </div>
            <div class="form-group">
                <input type="text" name="TxtFiltro" id="TxtFiltro" class="form-control" placeholder="Filtro: Nome" />
            </div>
            <button type="button" name="BtnPesquisar" id="BtnPesquisar" class="btn btn-info">Pesquisar</button>
        </div>        
    @Html.DataTablesHtml("example", "table table-striped", false, "Id", "Name", "Editar")        
    </div>    
    <script>
        $(document).ready(function () {
            var table = $('#example').DataTable({
                "processing": true,
                "serverSide": true,
                "lengthChange": false,
                "searching": false,
                "ajax": {
                    "url": "@Url.Action("PeoplesToResult")",
                    "type": "POST",
                    "data": function (d) {
                        d.Filter = { Name: $("#SelectFiltro").val(), Value: $("#TxtFiltro").val() };
                    }
                },
                "pageLength": 10,
                "rowId": 'Id',
                "columns": [
                    { "data": "Id", "Name": "Id" },
                    { "data": "Name", "Name": "Name" },
                    {
                        "data": null,
                        "orderable": false,
                        "defaultContent": '',
                        "render": function (data, type, row) {
                            return '<a href="/Pagina/Edit/' + data.Id + '" class="btn btn-success btn-block">Alterar</a>';
                        }
                    }],
                "language": 
                    {
                        "sEmptyTable": "Nenhum registro encontrado",
                        "sInfo": "Mostrando de _START_ até _END_ de _TOTAL_ registros",
                        "sInfoEmpty": "Mostrando 0 até 0 de 0 registros",
                        "sInfoFiltered": "(Filtrados de _MAX_ registros)",
                        "sInfoPostFix": "",
                        "sInfoThousands": ".",
                        "sLengthMenu": "_MENU_ resultados por página",
                        "sLoadingRecords": "Carregando...",
                        "sProcessing": "Processando...",
                        "sZeroRecords": "Nenhum registro encontrado",
                        "sSearch": "Pesquisar",
                        "oPaginate": {
                            "sNext": "Próximo",
                            "sPrevious": "Anterior",
                            "sFirst": "Primeiro",
                            "sLast": "Último"
                        },
                        "oAria": {
                            "sSortAscending": ": Ordenar colunas de forma ascendente",
                            "sSortDescending": ": Ordenar colunas de forma descendente"
                        }
                    }
            });
            $("#BtnPesquisar").on("click", function () {
                if (!Verify())
                {
                    alert("Código inválido");
                    return false;
                }
                table.draw();
            });
            $("#TxtFiltro").on("keypress", function (e) {
                if (e.keyCode == 13) {
                    if (!Verify()) {
                        alert("Código inválido");
                        return false;
                    }
                    table.draw();
                }
            });
            Verify = function()
            {
                reDigits = /^\d+$/;

                console.log(reDigits.test($("#TxtFiltro").val()));

                if ($("#SelectFiltro").val() == "Id")
                {
                    
                    return !isNaN(parseInt($("#TxtFiltro").val())) && reDigits.test($("#TxtFiltro").val());
                }
                return true;
            }
        });
    </script>
</body>
</html>

```