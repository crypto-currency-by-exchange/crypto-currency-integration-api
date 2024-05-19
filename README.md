### [![NuGet](https://img.shields.io/nuget/v/ShareInvest.UPbit?label=ShareInvest.UPbit&style=plastic&logo=nuget&color=004880)](https://www.nuget.org/packages/ShareInvest.UPbit)
```C#
using Newtonsoft.Json;

using ShareInvest.UPbit;

using (var api = new Quotation())
{
    var market = await api.GetMarketAsync();

    var tickers = await api.GetTickerAsync(market);

    foreach (var ticker in tickers)
    {
        Console.WriteLine(ticker.Code);
    }
    using (var socket = new WebSocket())
    {
        socket.SendTicker += (sender, e) =>
        {
            Console.WriteLine(JsonConvert.SerializeObject(e.Ticker, Formatting.Indented));
        };
        await socket.ConnectAsync();

        var task = Task.Run(socket.ReceiveAsync);

        await socket.RequestAsync(new
        {
            ticket = Guid.NewGuid()
        }, new
        {
            type = "ticker",
            codes = tickers.Select(e => e.Code)
        });
        await task;
    }
}
```

### bithumb
```C#
using Newtonsoft.Json;

using ShareInvest.Bithumb;

using (var api = new Quotation())
{
    var market = await api.GetMarketAsync();

    var tickers = await api.GetTickerAsync(market);

    foreach (var ticker in tickers)
    {
        Console.WriteLine(ticker.Code);
    }
    using (var socket = new WebSocket())
    {
        socket.SendTicker += (sender, e) =>
        {
            Console.WriteLine(JsonConvert.SerializeObject(e.Ticker, Formatting.Indented));
        };
        await socket.ConnectAsync();

        var task = Task.Run(socket.ReceiveAsync);

        static string swapStr(string? str)
        {
            var parts = str?.Split('-');

            return string.Join('_', parts?[^1], parts?[0]);
        }

        var bithumb = from ticker in tickers
                      where !string.IsNullOrEmpty(ticker.Code)
                      select swapStr(ticker.Code);

        await socket.RequestAsync("ticker", [.. bithumb], "30M", "1H", "12H", "24H", "MID");

        await task;
    }
}
```