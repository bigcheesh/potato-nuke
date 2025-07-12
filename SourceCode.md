Source Code:

C# Console app

```
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Text.Json;
using System.Text;
using System.Threading.Tasks;

class Program
{
    private const string BotToken = ""; // Replace with your bot's token
    private const string SecretKey = ""; // Access Key
    private static string GuildId = ""; // Don't add anything nor remove.

    static async Task Main(string[] args)
    {
        Console.Clear();
        DisplayBigText("ORBIT", ConsoleColor.Cyan);

        Console.Write("\nEnter the access key: ");
        string accessKey = Console.ReadLine();

        if (accessKey != SecretKey)
        {
            Console.WriteLine("[-] Invalid key. Exiting.".ColorText(ConsoleColor.Red));
            return;
        }

        using HttpClient httpClient = new HttpClient();
        httpClient.DefaultRequestHeaders.Add("Authorization", $"Bot {BotToken}");

        while (true)
        {
            Console.Clear();
            string[] menu = new string[]
            {
                $"Current Server ID: {(string.IsNullOrEmpty(GuildId) ? "Not Set" : GuildId)}",
                "1. Set Server (Guild) ID",
                "2. Create Channels",
                "3. Delete All Channels",
                "4. Spam Message to All Text Channels",
                "5. Ban All Members",
                "6. Kick All Members",
                "7. Exit"
            };

            int centerX = Console.WindowWidth / 2;
            int centerY = Console.WindowHeight / 2 - 3;

            foreach (var line in menu)
            {
                Console.SetCursorPosition(centerX - line.Length / 2, centerY++);
                Console.WriteLine(line.ColorText(ConsoleColor.Green));
            }

            Console.SetCursorPosition(centerX - 10, centerY);
            Console.Write("Enter your choice: ");
            string choice = Console.ReadLine();

            switch (choice)
            {
                case "1":
                    Console.Write("Enter Server (Guild) ID: ");
                    GuildId = Console.ReadLine();
                    Console.WriteLine("[+] Server ID set!".ColorText(ConsoleColor.Cyan));
                    await Task.Delay(1500);
                    break;

                case "2":
                    if (CheckGuildId()) await CreateChannels(httpClient);
                    break;

                case "3":
                    if (CheckGuildId()) await DeleteAllChannels(httpClient);
                    break;

                case "4":
                    if (CheckGuildId()) await SpamMessageToAllTextChannels(httpClient);
                    break;

                case "5":
                    if (CheckGuildId()) await BanAllMembers(httpClient);
                    break;

                case "6":
                    if (CheckGuildId()) await KickAllMembers(httpClient);
                    break;

                case "7":
                    Console.WriteLine("[+] Exiting program.".ColorText(ConsoleColor.Green));
                    return;

                default:
                    Console.WriteLine("[-] Invalid choice.".ColorText(ConsoleColor.Red));
                    break;
            }

            Console.WriteLine("\nPress Enter to continue...");
            Console.ReadLine();
        }
    }

    private static bool CheckGuildId()
    {
        if (string.IsNullOrEmpty(GuildId))
        {
            Console.WriteLine("[-] Please set the Server ID first (Option 1).".ColorText(ConsoleColor.Red));
            return false;
        }
        return true;
    }

    private static void DisplayBigText(string text, ConsoleColor color)
    {
        string[] bigText = new string[]
        {
            " OOO   RRRR   BBBB   III  TTTTT ",
            "O   O  R   R  B   B   I     T   ",
            "O   O  RRRR   BBBB    I     T   ",
            "O   O  R  R   B   B   I     T   ",
            " OOO   R   R  BBBB   III    T   "
        };

        Console.ForegroundColor = color;
        foreach (var line in bigText) Console.WriteLine(line);
        Console.ResetColor();
    }

    private static async Task CreateChannels(HttpClient httpClient)
    {
        Console.Write("How many channels to create? ");
        if (!int.TryParse(Console.ReadLine(), out int count) || count <= 0)
        {
            Console.WriteLine("[-] Invalid number.".ColorText(ConsoleColor.Red));
            return;
        }

        Console.Write("Enter the base name for the channels: ");
        string name = Console.ReadLine();

        var tasks = new List<Task>();
        for (int i = 1; i <= count; i++)
        {
            var payload = new { name = $"{name}-{i}", type = 0 };
            var content = new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json");
            tasks.Add(httpClient.PostAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels", content));
        }

        await Task.WhenAll(tasks);
        Console.WriteLine("[+] Channels created!".ColorText(ConsoleColor.Green));
    }

    private static async Task DeleteAllChannels(HttpClient httpClient)
    {
        Console.Write("Are you sure you want to delete ALL channels? (y/n): ");
        if (Console.ReadLine()?.ToLower() != "y")
        {
            Console.WriteLine("[-] Cancelled.".ColorText(ConsoleColor.Red));
            return;
        }

        var response = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels");
        var json = await response.Content.ReadAsStringAsync();
        var channels = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        var tasks = new List<Task>();
        foreach (var channel in channels)
        {
            string id = channel.GetProperty("id").GetString();
            tasks.Add(httpClient.DeleteAsync($"https://discord.com/api/v10/channels/{id}"));
        }

        await Task.WhenAll(tasks);
        Console.WriteLine("[+] Channels deleted!".ColorText(ConsoleColor.Green));
    }

    private static async Task SpamMessageToAllTextChannels(HttpClient httpClient)
    {
        Console.Write("Enter the message to spam: ");
        string message = Console.ReadLine();

        Console.Write("How many times per channel? ");
        if (!int.TryParse(Console.ReadLine(), out int count) || count <= 0)
        {
            Console.WriteLine("[-] Invalid number.".ColorText(ConsoleColor.Red));
            return;
        }

        var response = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels");
        var json = await response.Content.ReadAsStringAsync();
        var channels = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        var textChannels = new List<string>();
        foreach (var ch in channels)
            if (ch.GetProperty("type").GetInt32() == 0)
                textChannels.Add(ch.GetProperty("id").GetString());

        var tasks = new List<Task>();
        foreach (var ch in textChannels)
        {
            for (int i = 0; i < count; i++)
            {
                var payload = new { content = message };
                var content = new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json");
                tasks.Add(httpClient.PostAsync($"https://discord.com/api/v10/channels/{ch}/messages", content));
            }
        }

        await Task.WhenAll(tasks);
        Console.WriteLine($"[+] Spammed {count}x per text channel.".ColorText(ConsoleColor.Green));
    }

    private static async Task BanAllMembers(HttpClient httpClient)
    {
        var response = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/members?limit=1000");
        if (!response.IsSuccessStatusCode)
        {
            Console.WriteLine("[-] Failed to get members.".ColorText(ConsoleColor.Red));
            return;
        }

        string json = await response.Content.ReadAsStringAsync();
        var members = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        var tasks = new List<Task>();
        foreach (var member in members)
        {
            string userId = member.GetProperty("user").GetProperty("id").GetString();
            tasks.Add(httpClient.PutAsync($"https://discord.com/api/v10/guilds/{GuildId}/bans/{userId}", null));
        }

        await Task.WhenAll(tasks);
        Console.WriteLine("[+] All members banned.".ColorText(ConsoleColor.Green));
    }

    private static async Task KickAllMembers(HttpClient httpClient)
    {
        var response = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/members?limit=1000");
        if (!response.IsSuccessStatusCode)
        {
            Console.WriteLine("[-] Failed to get members.".ColorText(ConsoleColor.Red));
            return;
        }

        string json = await response.Content.ReadAsStringAsync();
        var members = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        var tasks = new List<Task>();
        foreach (var member in members)
        {
            string userId = member.GetProperty("user").GetProperty("id").GetString();
            tasks.Add(httpClient.DeleteAsync($"https://discord.com/api/v10/guilds/{GuildId}/members/{userId}"));
        }

        await Task.WhenAll(tasks);
        Console.WriteLine("[+] All members kicked.".ColorText(ConsoleColor.Green));
    }
}

static class ConsoleExtensions
{
    public static string ColorText(this string text, ConsoleColor color)
    {
        Console.ForegroundColor = color;
        Console.Write(text);
        Console.ResetColor();
        return "";
    }
}
```
