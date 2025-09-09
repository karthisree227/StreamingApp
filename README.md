// StreamingAppMain.java
import java.util.*;
import java.util.stream.Collectors;

/*
Console-based Video Streaming app demo:
- Classes: User, Plan, Content (abstract), Movie, Series, StreamingService
- Demonstrates: inheritance, method overloading, method overriding,
  polymorphism (List<Content> & calling overridden play()), encapsulation
*/

class Plan {
    private String planId;
    private String name;
    private double monthlyPrice;
    private int screens;
    private String quality; // e.g., "SD","HD","4K"

    // static variable: tracking all plans created
    public static List<Plan> allPlans = new ArrayList<>();

    public Plan(String planId, String name, double monthlyPrice, int screens, String quality) {
        this.planId = planId;
        this.name = name;
        this.monthlyPrice = monthlyPrice;
        this.screens = screens;
        this.quality = quality;
        allPlans.add(this);
    }

    // ≥5 methods including getters/setters
    public String getPlanId() { return planId; }
    public String getName() { return name; }
    public double getMonthlyPrice() { return monthlyPrice; }
    public int getScreens() { return screens; }
    public String getQuality() { return quality; }

    public void setName(String name) { this.name = name; }
    public void setMonthlyPrice(double monthlyPrice) { this.monthlyPrice = monthlyPrice; }
    public void setScreens(int screens) { this.screens = screens; }
    public void setQuality(String quality) { this.quality = quality; }

    @Override
    public String toString() {
        return String.format("%s (%s) - ₹%.2f /mo, %d screens, %s", name, planId, monthlyPrice, screens, quality);
    }
}

class User {
    private String userId;
    private String name;
    private String email;
    private Plan activePlan; // encapsulated
    private List<Content> watchlist;
    private List<Content> watchHistory;
    private boolean isActive; // account active flag

    // static variable to store all users
    public static List<User> allUsers = new ArrayList<>();

    public User(String userId, String name, String email) {
        this.userId = userId;
        this.name = name;
        this.email = email;
        this.activePlan = null;
        this.watchlist = new ArrayList<>();
        this.watchHistory = new ArrayList<>();
        this.isActive = true;
        allUsers.add(this);
    }

    // ≥5 methods (getters/setters and actions)
    public String getUserId() { return userId; }
    public String getName() { return name; }
    public String getEmail() { return email; }
    public Plan getActivePlan() { return activePlan; }
    public boolean isActive() { return isActive; }

    public void setName(String name) { this.name = name; }
    public void setEmail(String email) { this.email = email; }
    // plan changes are guarded through service (encapsulation). Provide only package/private setPlan:
    void setActivePlan(Plan plan) { this.activePlan = plan; }

    public void deactivate() { this.isActive = false; }
    public void activate() { this.isActive = true; }

    // watchlist/history operations
    public void addToWatchlist(Content c) {
        if (!watchlist.contains(c)) {
            watchlist.add(c);
            System.out.println(name + " added '" + c.getTitle() + "' to watchlist.");
        }
    }

    public void removeFromWatchlist(Content c) {
        watchlist.remove(c);
    }

    public List<Content> getWatchlist() { return Collections.unmodifiableList(watchlist); }
    public List<Content> getWatchHistory() { return Collections.unmodifiableList(watchHistory); }

    public void recordWatch(Content c) {
        watchHistory.add(c);
    }

    @Override
    public String toString() {
        return String.format("%s (%s) - Plan: %s", name, userId, activePlan == null ? "None" : activePlan.getName());
    }
}

abstract class Content {
    private String contentId;
    private String title;
    private String genre;
    private int year;
    private double rating; // average rating
    private int ratingCount;

    protected String type; // Movie or Series

    // static variable for catalog-wide uses
    public static List<Content> catalog = new ArrayList<>();

    public Content(String contentId, String title, String genre, int year, double initialRating) {
        this.contentId = contentId;
        this.title = title;
        this.genre = genre;
        this.year = year;
        this.rating = initialRating;
        this.ratingCount = initialRating > 0 ? 1 : 0;
        catalog.add(this);
    }

    // ≥4 instance variables already present above
    public String getContentId() { return contentId; }
    public String getTitle() { return title; }
    public String getGenre() { return genre; }
    public int getYear() { return year; }
    public double getRating() { return rating; }
    public String getType() { return type; }

    // Encapsulation for rating updates
    public void updateRating(double newRating) {
        if (newRating < 0 || newRating > 10) {
            System.out.println("Rating must be between 0 and 10.");
            return;
        }
        double total = this.rating * ratingCount;
        total += newRating;
        ratingCount++;
        this.rating = total / ratingCount;
        System.out.printf("Updated rating for '%s' → %.2f (from %d ratings)\n", title, rating, ratingCount);
    }

    // abstract play method to be overridden
    public abstract void play(User user);

    @Override
    public String toString() {
        return String.format("%s [%s] (%d) - %s - Rating: %.2f", title, contentId, year, genre, rating);
    }
}

class Movie extends Content {
    private int durationMinutes;
    private String director;
    private boolean isExclusive;

    public Movie(String contentId, String title, String genre, int year, double initialRating,
                 int durationMinutes, String director, boolean isExclusive) {
        super(contentId, title, genre, year, initialRating);
        this.durationMinutes = durationMinutes;
        this.director = director;
        this.isExclusive = isExclusive;
        this.type = "Movie";
    }

    // ≥5 methods (getters/setters + play + extra)
    public int getDurationMinutes() { return durationMinutes; }
    public String getDirector() { return director; }
    public boolean isExclusive() { return isExclusive; }
    public void setExclusive(boolean val) { this.isExclusive = val; }
    public void setDurationMinutes(int m) { this.durationMinutes = m; }

    // Method overriding: play behavior for Movie
    @Override
    public void play(User user) {
        System.out.println(user.getName() + " is playing Movie: " + getTitle() + " [" + durationMinutes + " mins]");
        user.recordWatch(this);
    }
}

class Series extends Content {
    private int episodes;
    private int avgEpisodeDuration;
    private String showrunner;
    // track last watched episode per user could be more complex; simplified approach below
    private Map<String, Integer> userLastEpisode = new HashMap<>();

    public Series(String contentId, String title, String genre, int year, double initialRating,
                  int episodes, int avgEpisodeDuration, String showrunner) {
        super(contentId, title, genre, year, initialRating);
        this.episodes = episodes;
        this.avgEpisodeDuration = avgEpisodeDuration;
        this.showrunner = showrunner;
        this.type = "Series";
    }

    public int getEpisodes() { return episodes; }
    public int getAvgEpisodeDuration() { return avgEpisodeDuration; }
    public String getShowrunner() { return showrunner; }
    public void setShowrunner(String s) { this.showrunner = s; }

    // Overriding: play default plays from user's last watched + 1
    @Override
    public void play(User user) {
        int last = userLastEpisode.getOrDefault(user.getUserId(), 0);
        int next = Math.min(last + 1, episodes);
        play(user, next);
    }

    // Overloaded play: play specific episode
    public void play(User user, int episodeNumber) {
        if (episodeNumber < 1 || episodeNumber > episodes) {
            System.out.println("Invalid episode number for " + getTitle());
            return;
        }
        System.out.println(user.getName() + " is watching Series: " + getTitle() + " — Episode " + episodeNumber +
                           " (" + avgEpisodeDuration + " mins)");
        userLastEpisode.put(user.getUserId(), episodeNumber);
        user.recordWatch(this);
    }
}

class StreamingService {
    private String name;
    private Map<String, User> users;
    private Map<String, Plan> plans;
    private Map<String, Content> contents;

    // play counts across catalog (contentId -> plays)
    private Map<String, Integer> playCounts;

    public StreamingService(String name) {
        this.name = name;
        this.users = new HashMap<>();
        this.plans = new HashMap<>();
        this.contents = new HashMap<>();
        this.playCounts = new HashMap<>();
    }

    // manage users/plans/content
    public void addPlan(Plan p) { plans.put(p.getPlanId(), p); }
    public void addUser(User u) { users.put(u.getUserId(), u); }
    public void addContent(Content c) { contents.put(c.getContentId(), c); }

    // subscribe user to plan (encapsulation: validate)
    public boolean subscribe(User user, Plan plan) {
        if (user == null || plan == null) return false;
        // Example guard: if user already on same plan
        if (plan.equals(user.getActivePlan())) {
            System.out.println(user.getName() + " is already on plan " + plan.getName());
            return false;
        }
        user.setActivePlan(plan);
        System.out.println(user.getName() + " subscribed to " + plan.getName());
        return true;
    }

    // change plan through service (guarded)
    public boolean changePlan(User user, Plan newPlan) {
        if (user == null || newPlan == null) return false;
        if (!user.isActive()) {
            System.out.println("Cannot change plan: user account inactive.");
            return false;
        }
        user.setActivePlan(newPlan);
        System.out.println("Plan changed for " + user.getName() + " → " + newPlan.getName());
        return true;
    }

    // play content polymorphically: accepts Content reference and invokes overridden play
    public void playContent(User user, Content c) {
        if (user == null || c == null) return;
        // simulate play
        c.play(user); // polymorphism: runtime calls Movie.play() or Series.play()
        // update play counts
        playCounts.put(c.getContentId(), playCounts.getOrDefault(c.getContentId(), 0) + 1);
    }

    // add to watchlist
    public void addToWatchlist(User user, Content c) {
        if (user == null || c == null) return;
        user.addToWatchlist(c);
    }

    // Recommendations: method overloading by genre / minRating / year
    public List<Content> recommend(User user) {
        // simple strategy: recommend top-rated not in history
        Set<String> watchedIds = user.getWatchHistory().stream()
                                     .map(Content::getContentId).collect(Collectors.toSet());
        return contents.values().stream()
                .filter(c -> !watchedIds.contains(c.getContentId()))
                .sorted((a,b) -> Double.compare(b.getRating(), a.getRating()))
                .limit(5).collect(Collectors.toList());
    }

    public List<Content> recommend(User user, String genre) {
        Set<String> watchedIds = user.getWatchHistory().stream()
                                     .map(Content::getContentId).collect(Collectors.toSet());
        return contents.values().stream()
                .filter(c -> c.getGenre().equalsIgnoreCase(genre) && !watchedIds.contains(c.getContentId()))
                .sorted((a,b) -> Double.compare(b.getRating(), a.getRating()))
                .limit(5).collect(Collectors.toList());
    }

    public List<Content> recommend(User user, int minYear, double minRating) {
        Set<String> watchedIds = user.getWatchHistory().stream()
                                     .map(Content::getContentId).collect(Collectors.toSet());
        return contents.values().stream()
                .filter(c -> c.getYear() >= minYear && c.getRating() >= minRating && !watchedIds.contains(c.getContentId()))
                .sorted((a,b) -> Double.compare(b.getRating(), a.getRating()))
                .limit(10).collect(Collectors.toList());
    }

    // Analytics helpers
    public List<Content> topWatched(int topN) {
        List<Map.Entry<String,Integer>> list = new ArrayList<>(playCounts.entrySet());
        list.sort((a,b) -> Integer.compare(b.getValue(), a.getValue()));
        List<Content> result = new ArrayList<>();
        for (int i=0; i<Math.min(topN, list.size()); i++) {
            String cid = list.get(i).getKey();
            result.add(contents.get(cid));
        }
        return result;
    }

    public Map<String, Double> planWiseRevenue() {
        // monthly revenue = sum(planPrice for each user)
        Map<String, Double> revenue = new HashMap<>();
        for (User u : users.values()) {
            Plan p = u.getActivePlan();
            if (p != null) {
                revenue.put(p.getName(), revenue.getOrDefault(p.getName(), 0.0) + p.getMonthlyPrice());
            }
        }
        return revenue;
    }

    // utility to print service state
    public void printCatalog() {
        System.out.println("Catalog:");
        contents.values().forEach(System.out::println);
    }

    public void printUsers() {
        System.out.println("Users:");
        users.values().forEach(System.out::println);
    }
}

public class StreamingAppMain {
    public static void main(String[] args) {
        StreamingService service = new StreamingService("Streamly");

        // Create Plans
        Plan basic = new Plan("P1", "Basic", 199.0, 1, "SD");
        Plan standard = new Plan("P2", "Standard", 499.0, 2, "HD");
        Plan premium = new Plan("P3", "Premium", 799.0, 4, "4K");

        service.addPlan(basic);
        service.addPlan(standard);
        service.addPlan(premium);

        // Create Users
        User alice = new User("U1", "Alice", "alice@example.com");
        User bob = new User("U2", "Bob", "bob@example.com");
        User charlie = new User("U3", "Charlie", "charlie@example.com");

        service.addUser(alice);
        service.addUser(bob);
        service.addUser(charlie);

        // Subscribe users to plans
        service.subscribe(alice, premium);
        service.subscribe(bob, standard);
        service.subscribe(charlie, basic);

        // Create Contents
        Movie m1 = new Movie("C1", "The Last Dawn", "Sci-Fi", 2022, 8.5, 130, "R. Kaur", false);
        Movie m2 = new Movie("C2", "Sunny Days", "Romance", 2021, 7.2, 105, "S. Patel", true);
        Series s1 = new Series("C3", "Mystery Mansion", "Thriller", 2023, 9.0, 8, 45, "A. Kapoor");
        Series s2 = new Series("C4", "Slice of Life", "Drama", 2020, 8.0, 12, 30, "M. Rao");

        service.addContent(m1);
        service.addContent(m2);
        service.addContent(s1);
        service.addContent(s2);

        // Users build watchlists
        service.addToWatchlist(alice, m1);
        service.addToWatchlist(alice, s1);
        service.addToWatchlist(bob, m2);
        service.addToWatchlist(charlie, s2);

        // Play content (polymorphism + overriding)
        service.playContent(alice, m1);   // Movie.play()
        service.playContent(alice, s1);   // Series.play() -> plays ep 1
        // Play a specific episode (Series overloaded play)
        s1.play(bob, 3);                  // direct call to overloaded play(ep)
        service.playContent(bob, m2);
        // Play same content multiple times to create top-watched
        service.playContent(alice, m1);
        service.playContent(charlie, s2);
        service.playContent(charlie, s2);

        // Update ratings (encapsulation)
        m1.updateRating(9.0);
        s1.updateRating(8.8);

        // Print users and catalog
        System.out.println();
        service.printUsers();
        System.out.println();
        service.printCatalog();

        // Top-watched
        System.out.println("\nTop Watched:");
        List<Content> top = service.topWatched(3);
        for (Content c : top) {
            System.out.println(" - " + c.getTitle() + " (Plays: see playCounts internal)");
        }

        // Plan-wise revenue
        System.out.println("\nPlan-wise Monthly Revenue:");
        Map<String, Double> rev = service.planWiseRevenue();
        rev.forEach((k,v) -> System.out.printf(" - %s : ₹%.2f\n", k, v));

        // Personalized recommendations
        System.out.println("\nRecommendations for Alice (general):");
        List<Content> recAlice = service.recommend(alice);
        recAlice.forEach(c -> System.out.println("  * " + c));

        System.out.println("\nRecommendations for Bob (genre: Thriller):");
        List<Content> recBob = service.recommend(bob, "Thriller");
        recBob.forEach(c -> System.out.println("  * " + c));

        System.out.println("\nRecommendations for Charlie (since 2021, minRating 7.5):");
        List<Content> recCharlie = service.recommend(charlie, 2021, 7.5);
        recCharlie.forEach(c -> System.out.println("  * " + c));

        // Demonstrate polymorphism on List<Content>
        System.out.println("\nPolymorphism demo: Play a mixed list polymorphically:");
        List<Content> mixed = Arrays.asList(m1, s2, m2, s1);
        for (Content c : mixed) {
            // Note: play will use runtime type (Movie/Series) implementations
            service.playContent(alice, c);
        }

        // Final summary: print Alice watch history
        System.out.println("\nAlice watch history:");
        for (Content c : alice.getWatchHistory()) {
            System.out.println(" - " + c.getTitle() + " (" + c.getType() + ")");
        }
    }
}
