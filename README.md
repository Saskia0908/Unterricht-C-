# Unterricht-C-
Programmierung
	namespace PersonalVerwaltung
	using System;
using System;
using System.Collections.Generic;
using System.Collections.Generic;
using System.Globalization;
using System.IO;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

	{
		public class Mitarbeiter
		{
			public Guid Id { get; set; } = Guid.NewGuid();
			public string Vorname { get; set; } = "";
			public string Nachname { get; set; } = "";
			public int Urlaubstage { get; set; } = 0;
			public List<DateTime> Krankheitstage { get; set; } = new List<DateTime>();

			public override string ToString()
			{
				string krank;
				if (Krankheitstage == null || Krankheitstage.Count == 0)
				{
					krank = "(keine)";
				}
				else
				{
					krank = string.Join(", ", Krankheitstage.Select(d => d.ToString("dd.MM.yyyy", CultureInfo.InvariantCulture)));
				}

				return $"Id: {Id}\nName: {Vorname} {Nachname}\nUrlaubstage: {Urlaubstage}\nKrankheitstage: {krank}\n";
			}
		}

		public static class Speicher
		{
			private static readonly string DateiPfad = Path.Combine(AppContext.BaseDirectory, "employees.json");

			private static readonly JsonSerializerOptions Options = new JsonSerializerOptions
			{
				WriteIndented = true,
				PropertyNameCaseInsensitive = true
			};

			public static List<Mitarbeiter> Laden()
			{
				try
				{
					if (!File.Exists(DateiPfad))
					{
						return new List<Mitarbeiter>();
					}

					string json = File.ReadAllText(DateiPfad, Encoding.UTF8);
					var list = JsonSerializer.Deserialize<List<Mitarbeiter>>(json, Options);
					return list ?? new List<Mitarbeiter>();
				}
				catch (Exception ex)
				{
					Console.WriteLine("Fehler beim Laden der Datei: " + ex.Message);
					return new List<Mitarbeiter>();
				}
			}

			public static bool Speichern(List<Mitarbeiter> mitarbeiter)
			{
				try
				{
					string json = JsonSerializer.Serialize(mitarbeiter, Options);
					File.WriteAllText(DateiPfad, json, Encoding.UTF8);
					return true;
				}
				catch (Exception ex)
				{
					Console.WriteLine("Fehler beim Speichern der Datei: " + ex.Message);
					return false;
				}
			}
		}

		class Program
		{
			static List<Mitarbeiter> mitarbeiterListe = new List<Mitarbeiter>();

			static void Main(string[] args)
			{
				Console.OutputEncoding = Encoding.UTF8;
				Console.WriteLine("=== Personalverwaltungsprogramm ===");
				mitarbeiterListe = Speicher.Laden();
				Console.WriteLine($"Geladene Datensätze: {mitarbeiterListe.Count}");
				MenuLoop();
			}

			static void MenuLoop()
			{
				while (true)
				{
					Console.WriteLine("\nMenü:");
					Console.WriteLine("1) Alle Mitarbeiter anzeigen");
					Console.WriteLine("2) Mitarbeiter hinzufügen");
					Console.WriteLine("3) Mitarbeiter entfernen");
					Console.WriteLine("4) Krankheitstag hinzufügen");
					Console.WriteLine("5) Krankheitstag entfernen");
					Console.WriteLine("6) Mitarbeiter bearbeiten (Urlaubstage ändern)");
					Console.WriteLine("0) Beenden");
					Console.Write("Wahl: ");
					string input = Console.ReadLine() ?? "";

					switch (input.Trim())
					{
						case "1":
							ListAlle();
							break;
						case "2":
							Hinzufuegen();
							break;
						case "3":
							Entfernen();
							break;
						case "4":
							KrankheitstagHinzufuegen();
							break;
						case "5":
							KrankheitstagEntfernen();
							break;
						case "6":
							Bearbeiten();
							break;
						case "0":
							Beenden();
							return;
						default:
							Console.WriteLine("Ungültige Eingabe, bitte erneut.");
							break;
					}
				}
			}

			static void ListAlle()
			{
				if (mitarbeiterListe.Count == 0)
				{
					Console.WriteLine("Keine Mitarbeiter vorhanden.");
					return;
				}

				Console.WriteLine("\n--- Mitarbeiterliste ---");
				for (int i = 0; i < mitarbeiterListe.Count; i++)
				{
					var m = mitarbeiterListe[i];
					Console.WriteLine($"[{i + 1}] {m.Vorname} {m.Nachname} (Id: {m.Id})");
				}

				Console.Write("Möchtest du Details zu einem Mitarbeiter sehen? (Index oder ENTER zum Zurück): ");
				string sel = Console.ReadLine() ?? "";
				if (string.IsNullOrWhiteSpace(sel)) return;

				if (int.TryParse(sel.Trim(), out int index) && index >= 1 && index <= mitarbeiterListe.Count)
				{
					Console.WriteLine();
					Console.WriteLine(mitarbeiterListe[index - 1].ToString());
				}
				else
				{
					Console.WriteLine("Ungültiger Index.");
				}
			}

			static void Hinzufuegen()
			{
				Console.WriteLine("\n--- Mitarbeiter hinzufügen ---");
				Console.Write("Vorname: ");
				string vorname = ReadNonEmptyString();
				Console.Write("Nachname: ");
				string nachname = ReadNonEmptyString();
				Console.Write("Urlaubstage (ganze Zahl, >=0): ");
				int urlaub = ReadInt(min: 0);

				var neu = new Mitarbeiter
				{
					Vorname = vorname,
					Nachname = nachname,
					Urlaubstage = urlaub
				};

				mitarbeiterListe.Add(neu);
				if (Speicher.Speichern(mitarbeiterListe))
					Console.WriteLine("Mitarbeiter hinzugefügt und gespeichert.");
				else
					Console.WriteLine("Mitarbeiter hinzugefügt, aber Speichern fehlgeschlagen.");
			}

			static void Entfernen()
			{
				if (mitarbeiterListe.Count == 0)
				{
					Console.WriteLine("Keine Mitarbeiter zum Entfernen vorhanden.");
					return;
				}

				Console.WriteLine("\n--- Mitarbeiter entfernen ---");
				for (int i = 0; i < mitarbeiterListe.Count; i++)
				{
					Console.WriteLine($"[{i + 1}] {mitarbeiterListe[i].Vorname} {mitarbeiterListe[i].Nachname}");
				}

				Console.Write("Index des zu entfernenden Mitarbeiters: ");
				string selection = Console.ReadLine() ?? "";
				if (int.TryParse(selection.Trim(), out int idx) && idx >= 1 && idx <= mitarbeiterListe.Count)
				{
					var entfernt = mitarbeiterListe[idx - 1];
					Console.Write($"Willst du '{entfernt.Vorname} {entfernt.Nachname}' wirklich entfernen? (j/n): ");
					string confirm = (Console.ReadLine() ?? "").Trim().ToLowerInvariant();
					if (confirm.StartsWith("j"))
					{
						mitarbeiterListe.RemoveAt(idx - 1);
						if (Speicher.Speichern(mitarbeiterListe))
							Console.WriteLine("Mitarbeiter entfernt und gespeichert.");
						else
							Console.WriteLine("Mitarbeiter entfernt, aber Speichern fehlgeschlagen.");
					}
					else
					{
						Console.WriteLine("Abgebrochen.");
					}
				}
				else
				{
					Console.WriteLine("Ungültige Eingabe.");
				}
			}

			static void KrankheitstagHinzufuegen()
			{
				if (mitarbeiterListe.Count == 0)
				{
					Console.WriteLine("Keine Mitarbeiter vorhanden.");
					return;
				}

				Console.WriteLine("\n--- Krankheitstag hinzufügen ---");
				var mit = WaehleMitarbeiter();
				if (mit == null) return;

				Console.Write("Datum des Krankheitstags (dd.MM.yyyy oder yyyy-MM-dd): ");
				if (ReadDate(out DateTime dt))
				{
					dt = dt.Date;
					if (mit.Krankheitstage.Any(d => d.Date == dt))
					{
						Console.WriteLine("Dieser Krankheitstag ist bereits eingetragen.");
					}
					else
					{
						mit.Krankheitstage.Add(dt);
						mit.Krankheitstage.Sort();
						if (Speicher.Speichern(mitarbeiterListe))
							Console.WriteLine("Krankheitstag hinzugefügt und gespeichert.");
						else
							Console.WriteLine("Krankheitstag hinzugefügt, aber Speichern fehlgeschlagen.");
					}
				}
				else
				{
					Console.WriteLine("Ungültiges Datum.");
				}
			}

			static void KrankheitstagEntfernen()
			{
				if (mitarbeiterListe.Count == 0)
				{
					Console.WriteLine("Keine Mitarbeiter vorhanden.");
					return;
				}

				Console.WriteLine("\n--- Krankheitstag entfernen ---");
				var mit = WaehleMitarbeiter();
				if (mit == null) return;

				if (mit.Krankheitstage == null || mit.Krankheitstage.Count == 0)
				{
					Console.WriteLine("Keine Krankheitstage eingetragen.");
					return;
				}

				Console.WriteLine("Krankheitstage:");
				for (int i = 0; i < mit.Krankheitstage.Count; i++)
				{
					Console.WriteLine($"[{i + 1}] {mit.Krankheitstage[i]:dd.MM.yyyy}");
				}

				Console.Write("Index des zu entfernenden Krankheitstags: ");
				string sel = Console.ReadLine() ?? "";
				if (int.TryParse(sel.Trim(), out int idx) && idx >= 1 && idx <= mit.Krankheitstage.Count)
				{
					var removed = mit.Krankheitstage[idx - 1];
					mit.Krankheitstage.RemoveAt(idx - 1);
					if (Speicher.Speichern(mitarbeiterListe))
						Console.WriteLine($"Krankheitstag {removed:dd.MM.yyyy} entfernt und gespeichert.");
					else
						Console.WriteLine("Krankheitstag entfernt, aber Speichern fehlgeschlagen.");
				}
				else
				{
					Console.WriteLine("Ungültige Eingabe.");
				}
			}

			static void Bearbeiten()
			{
				if (mitarbeiterListe.Count == 0)
				{
					Console.WriteLine("Keine Mitarbeiter vorhanden.");
					return;
				}

				Console.WriteLine("\n--- Mitarbeiter bearbeiten (Urlaubstage) ---");
				var mit = WaehleMitarbeiter();
				if (mit == null) return;

				Console.WriteLine($"Aktuelle Urlaubstage: {mit.Urlaubstage}");
				Console.Write("Neue Anzahl Urlaubstage eingeben (ganze Zahl, >=0): ");
				int neu = ReadInt(min: 0);
				mit.Urlaubstage = neu;
				if (Speicher.Speichern(mitarbeiterListe))
					Console.WriteLine("Änderung gespeichert.");
				else
					Console.WriteLine("Änderung vorgenommen, aber Speichern fehlgeschlagen.");
			}

			static Mitarbeiter? WaehleMitarbeiter()
			{
				for (int i = 0; i < mitarbeiterListe.Count; i++)
				{
					Console.WriteLine($"[{i + 1}] {mitarbeiterListe[i].Vorname} {mitarbeiterListe[i].Nachname}");
				}
				Console.Write("Index wählen: ");
				string sel = Console.ReadLine() ?? "";
				if (int.TryParse(sel.Trim(), out int idx) && idx >= 1 && idx <= mitarbeiterListe.Count)
				{
					return mitarbeiterListe[idx - 1];
				}
				Console.WriteLine("Ungültige Eingabe.");
				return null;
			}

			static void Beenden()
			{
				Console.WriteLine("Beende Programm. Daten wurden bei jeder Änderung gespeichert.");
			}

			static string ReadNonEmptyString()
			{
				while (true)
				{
					string s = Console.ReadLine() ?? "";
					if (!string.IsNullOrWhiteSpace(s)) return s.Trim();
					Console.Write("Eingabe leer — bitte erneut: ");
				}
			}

			static int ReadInt(int min = int.MinValue, int max = int.MaxValue)
			{
				while (true)
				{
					string s = Console.ReadLine() ?? "";
					if (int.TryParse(s.Trim(), out int val) && val >= min && val <= max) return val;
					if (min == int.MinValue && max == int.MaxValue)
						Console.Write("Ungültige Zahl. Bitte erneut: ");
					else
						Console.Write($"Ungültige Zahl. Bitte eine ganze Zahl zwischen {min} und {max} eingeben: ");
				}
			}

			static bool ReadDate(out DateTime datum)
			{
				string s = Console.ReadLine() ?? "";
				string[] formats = new[] { "dd.MM.yyyy", "d.M.yyyy", "yyyy-MM-dd", "dd.MM.yy" };

				if (DateTime.TryParseExact(s.Trim(), formats, CultureInfo.InvariantCulture, DateTimeStyles.None, out datum))
				{
					datum = datum.Date;
					return true;
				}

				if (DateTime.TryParse(s.Trim(), CultureInfo.InvariantCulture, DateTimeStyles.None, out datum))
				{
					datum = datum.Date;
					return true;
				}

				datum = default;
				return false;
			}
		}
	}
}
