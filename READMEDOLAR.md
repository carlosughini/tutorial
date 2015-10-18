#Canducci DOLAR (QuoteDolar) 

###(version: 1.0.0)


__Web Service Yahoo http://viacep.com.br/__

##[Demo](http://canduccipackages.apphb.com/#/)

[![Canducci Dolar](http://i666.photobucket.com/albums/vv25/netdragoon/1430207215_money-increase-64_zps3sjc4h5j.png)](https://www.nuget.org/packages/CanducciQuoteDolar/)

##Instalação do Pacote (NUGET)

```Csharp
PM> Install-Package CanducciQuoteDolar
```

##Como utilizar?

Declare o namespace `using Canducci.QuoteDolar;` 


```Csharp

try
{
    using (Dolar dolar = new Dolar())
    {
        DolarInfo dolarInfo = dolar.DolarInfo();
        //Dolar no Brasil
        RatesInfo rateInfoUSDBRL = dolarInfo.RatesInfo.GetRatesInfo(RatesInfoType.USDBRL);

    }
}
catch (Exception ex)
{

    throw ex;
}

```