import java.util.*;
import java.util.stream.*;

public class Campeonato {

    // Classe Time
    static class Time {
        String nome;
        int pontos = 0;
        int vitorias = 0;
        int empates = 0;
        int derrotas = 0;
        int golsPro = 0;
        int golsContra = 0;

        public Time(String nome) {
            this.nome = nome;
        }

        public int saldoGols() {
            return golsPro - golsContra;
        }

        public String getCategoria() {
            int pts = pontos;

            switch (pts / 4) {
                case 5:
                    return "LÍDER";
                case 3:
                case 4:
                    return "G4";
                case 2:
                    return "MEIO DE TABELA";
                case 1:
                    return "ALERTA";
                default:
                    return "REBAIXAMENTO";
            }
        }
    }

    public static void main(String[] args) {

        String[] partidas = {
            "Flamengo:3:1:Palmeiras",
            "Corinthians:0:0:São Paulo",
            "Atletico-MG:2:2:Fluminense",
            "Palmeiras:1:0:Corinthians",
            "São Paulo:3:2:Flamengo",
            "Fluminense:0:1:Atletico-MG",
            "Flamengo:2:0:Corinthians",
            "Palmeiras:4:1:Fluminense",
            "São Paulo:0:0:Atletico-MG",
            "Corinthians:1:3:Fluminense",
            "Atletico-MG:0:2:Flamengo",
            "Fluminense:1:1:São Paulo"
        };

        Map<String, Time> tabela = new HashMap<>();

        // TAREFA 1 + 2
        for (String linha : partidas) {

            String[] dados = parsearPartida(linha);
            if (dados == null) continue;

            String casa = dados[0];
            int golsCasa = Integer.parseInt(dados[1]);
            int golsFora = Integer.parseInt(dados[2]);
            String fora = dados[3];

            tabela.putIfAbsent(casa, new Time(casa));
            tabela.putIfAbsent(fora, new Time(fora));

            Time tCasa = tabela.get(casa);
            Time tFora = tabela.get(fora);

            // gols
            tCasa.golsPro += golsCasa;
            tCasa.golsContra += golsFora;

            tFora.golsPro += golsFora;
            tFora.golsContra += golsCasa;

            // resultado
            if (golsCasa > golsFora) {
                tCasa.pontos += 3;
                tCasa.vitorias++;
                tFora.derrotas++;
            } else if (golsCasa < golsFora) {
                tFora.pontos += 3;
                tFora.vitorias++;
                tCasa.derrotas++;
            } else {
                tCasa.pontos++;
                tFora.pontos++;
                tCasa.empates++;
                tFora.empates++;
            }
        }

        List<Time> times = new ArrayList<>(tabela.values());

        Time maiorAtaque = times.stream()
                .max(Comparator.comparingInt(t -> t.golsPro))
                .orElse(null);

        System.out.println("\nMaior ataque: " + maiorAtaque.nome + " (" + maiorAtaque.golsPro + " gols)");

        double media = Arrays.stream(partidas)
                .mapToInt(p -> {
                    String[] d = p.split(":");
                    return Integer.parseInt(d[1]) + Integer.parseInt(d[2]);
                })
                .average()
                .orElse(0);

        System.out.println("Média de gols por partida: " + String.format("%.2f", media));

    
        System.out.println("\nTimes na zona de rebaixamento:");
        times.stream()
                .filter(t -> t.pontos < 4)
                .forEach(t -> System.out.println(t.nome));

       
        times = times.stream()
                .sorted((a, b) -> b.pontos - a.pontos)
                .collect(Collectors.toList());


        System.out.println("\n=== CAMPEONATO BRASILEIRO 2026 ===");
        System.out.println("POS | TIME | PTS | V | E | D | SG | CATEGORIA");
        System.out.println("-------------------------------------------------");

        int pos = 1;
        for (Time t : times) {
            System.out.println(String.format(
                    "%2d | %-12s | %3d | %2d | %2d | %2d | %+3d | %s",
                    pos++, t.nome, t.pontos, t.vitorias,
                    t.empates, t.derrotas, t.saldoGols(), t.getCategoria()
            ));
        }
    }

    public static String[] parsearPartida(String linha) {
        try {
            String[] partes = linha.split(":");

            if (partes.length != 4) {
                System.out.println("[ERRO] Linha inválida: " + linha);
                return null;
            }

            for (int i = 0; i < partes.length; i++) {
                partes[i] = partes[i].trim().toUpperCase();
            }

            return partes;

        } catch (Exception e) {
            System.out.println("[ERRO] Linha inválida: " + linha);
            return null;
        }
    }
}
