# BlogAspNet

Projeto para revisão de conceito e aprendizado,
continuação do projeto [BlogAspNet](https://github.com/thiagokj/BlogAspNet_Validations)

Alguns exemplos sobre Configurações e Serviços externos.

## AppSettings

O arquivo appsettings.json é utilizado para armazenar as configurações da aplicação em tempo
de execução. O mesmo pode ser separado em ambientes, como Desenvolvimento e Produção.

O AppSettings não deve conter dados sensíveis como credenciais e chaves de acesso. Informações secretas
devem ser armazenadas no servidor de forma segura utilizando **dotnet secrets**, por exemplo.

```Csharp
// Exemplo de uma classe de configuração, com classes internas para melhor organização do código.
public static class Configuration
{
    public static string JwtKey = "chaveJwt";
    public static string ApiKeyName = "nomeDaChave";
    public static string ApiKey = "chaveDaAPI";
    public string SmtpConfiguration Smtp = new ();

    public class SmtpConfiguration
    {
        public string Host { get; set; }
        public int Port { get; set; } = 25;
        public string UserName { get; set; }
        public string Password { get; set; }
    }
}

// Fazendo o vinculo dos nós do appsettings.json com as propriedades da classe Configuration
...
var app = builder.Build();

// Retorna os valores presentes no appsettings.json como string. 
app.Configuration.GetValue<string>("JwtKey");
app.Configuration.GetValue<string>("ApiKeyName");
app.Configuration.GetValue<string>("ApiKey");

// Retorna toda sessão do appsettings.json e faz o vinculo as propriedades da classe.
var smtp = new Configuration.SmtpConfiguration();
app.Configuration.GetSection("Smtp").Bind(smtp);
Configuration.Smtp = smtp;
```

```javascript
{
  "JwtKey": "minhaChaveJwt",
  "ApiKeyName": "api_key",
  "ApiKey": "curso_api_MinhaChave",
  "Smtp": {
    "Host": "smtp.sendgrid.net",
    "Port": "587",
    "UserName": "apikey",
    "Password": "suasenha"
  },
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*"
}
```

## Utilizando APIs externas

Podemos realizar o envio de emails por meio da API do SendGrid.

Obs: Na utilização da API do sendGrid com Smtp, o usuario deve ser **apikey** e o password
a chave gerada em sua conta no SendGrid.

Obs: Para cada novo serviço criado, registre o mesmo no builder.

```Csharp
...
void ConfigureServices(WebApplicationBuilder builder)
{
    builder.Services.AddDbContext<BlogDataContext>();
    builder.Services.AddTransient<TokenService>();
    builder.Services.AddTransient<EmailService>();
}
```

```Csharp
// Exemplo de serviço de email usando SMTP.
public class EmailService
{
    public bool Send(
        string toName,
        string toEmail,
        string subject,
        string body,
        string fromName = "Nome do seu time",
        string fromEmail = "email@seudominio.sufix")
    {
        var smtpClient = new SmtpClient(Configuration.Smtp.Host, Configuration.Smtp.Port);

        smtpClient.Credentials = new NetworkCredential(Configuration.Smtp.UserName,
            Configuration.Smtp.Password); 
        smtpClient.DeliveryMethod = SmtpDeliveryMethod.Network;
        smtpClient.EnableSsl = true;

        var mail = new MailMessage();

        mail.From = new MailAddress(fromEmail, fromName);
        mail.To.Add(new MailAddress(toEmail, toName));
        mail.Subject = subject;
        mail.Body = body;
        mail.IsBodyHtml = true;

        try
        {
            smtpClient.Send(mail);
            return true;
        }
        catch (Exception)
        {
            return false;
        }
    }
}
```
Com o serviço criado e registrado, adicione o email ao AccountController.

# Upload de arquivos

Para permitir o servidor renderizar imagens e arquivos (html, jpg, js, etc) habilite no
builder o **UseStaticFiles()**.

```Csharp
app.UseAuthentication();
app.UseAuthorization();
app.UseStaticFiles(); // Essa configuração busca a pasta **wwwroot** da aplicação.
app.MapControllers();
app.Run();
```

Evite manter arquivos estáticos dentro da API, pois isso deixa a aplicação
mais pesada. A cada response o servidor ficará ocupado até devolver esses arquivos.

O ideal é armazenar em storages na nuvem, como Azure, e guardar apenas o caminho.

Aproveite parar organizar as pastas da aplicação, criando subpastas e agrupando as
pastas relacionadas. Note que os namespaces devem ser atualizados para não quebrar
os vinculos.
