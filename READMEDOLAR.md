#Canducci DOLAR (QuoteDolar) 

###(version: 1.0.0)


__Web Service Yahoo__

##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Dolar](http://i666.photobucket.com/albums/vv25/netdragoon/1430207215_money-increase-64_zps3sjc4h5j.png)](https://www.nuget.org/packages/CanducciQuoteDolar/)

[![NuGet](https://img.shields.io/nuget/dt/CanducciQuoteDolar.svg?style=plastic&label=downloads)](https://www.nuget.org/packages/CanducciQuoteDolar/)
[![NuGet](https://img.shields.io/nuget/v/CanducciQuoteDolar.svg?style=plastic&label=version)](https://www.nuget.org/packages/CanducciQuoteDolar/)

##Instalação do Pacote (NUGET)

```Csharp

PM> Install-Package CanducciQuoteDolar

```

##Como utilizar?

_Declare o namespace_ `using Canducci.QuoteDolar;`

####Version 2.0.0

1 - Simples
```Csharp
Dolar dolar = new Dolar();
DolarInfo info = dolar.DolarInfo();
RatesInfo rate = info.RatesInfo.GetRatesInfoUSDBRL();
```

2 - Async (Version NET = 4.0)
```Csharp
static void Main(string[] args)
{
    Dolar dolar = new Dolar();
    dolar.DolarInfoAsyncCompleted += Dolar_DolarInfoAsyncCompleted;
    dolar.DolarInfoAsync();            

    Console.ReadKey();
}

private static void Dolar_DolarInfoAsyncCompleted(DolarInfo dolarInfo)
{

    Console.WriteLine(dolarInfo.Count);            
    Console.WriteLine(dolarInfo.Created);
    Console.WriteLine(dolarInfo.Lang);
    

    dolarInfo.RatesInfo.ToList().ForEach(x =>
    {
        Console.WriteLine("*********************************");
        Console.WriteLine(x.Id);
        Console.WriteLine(x.Ask);
        Console.WriteLine(x.Bid);
        Console.WriteLine(x.Date);
        Console.WriteLine(x.Rate);
        Console.WriteLine(x.Time);                
    });
    Console.WriteLine("*********************************");
}
```

3 - Async (Version NET >= 4.5)

__Example MVC ASP.NET__

```Csharp
public class ExampleController : Controller
{
    private readonly Dolar _dolar;

    public HomeController()
    {
        _dolar = new Dolar();
    }

    protected override void Dispose(bool disposing)
    {
        if (_dolar != null)
        {
            _dolar.Dispose();
        }
        base.Dispose(disposing);
    }

    public async Task<ActionResult> DolarInfoAsync()
    {        

        DolarInfo _info = await _dolar.DolarInfoAsync();
        
        RatesInfo _result = _info.RatesInfo.GetRatesInfoUSDBRL();

        return View(_result);
    }
```
---

####Version 1.0.0

```Csharp

try
{
    using (Dolar dolar = new Dolar())
    {
        DolarInfo dolarInfo = dolar.DolarInfo();
        //Dolar no Brasil
        RatesInfo rateInfoUSDBRL = dolarInfo.RatesInfo.GetRatesInfo(RatesInfoType.USDBRL);
        //rateInfoUSDBRL.Bid (Compra)
        //rateInfoUSDBRL.Ask (Venda)
    }
}
catch (Exception ex)
{

    throw ex;
}

```