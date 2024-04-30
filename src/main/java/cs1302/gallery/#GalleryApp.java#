package cs1302.gallery;

import java.net.http.HttpClient;
import javafx.application.Application;
import javafx.application.Platform;
import javafx.scene.layout.HBox;
import javafx.scene.Scene;
import javafx.stage.Stage;
import javafx.scene.image.ImageView;
import javafx.scene.image.Image;
import javafx.event.EventHandler;
import javafx.scene.control.ComboBox;
import javafx.scene.layout.Region;
import java.io.IOException;
import javafx.scene.layout.VBox;
import javafx.scene.control.ProgressBar;
import javafx.scene.control.Label;
import javafx.scene.control.Button;
import javafx.scene.control.TextField;
import javafx.animation.Timeline;
import javafx.animation.KeyFrame;
import java.util.ArrayList;
import java.nio.charset.StandardCharsets;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.net.http.HttpClient;
import javafx.scene.control.Alert;
import javafx.util.Duration;
import java.net.URLEncoder;
import java.util.List;
import javafx.scene.layout.Priority;
import java.net.URI;
import javafx.scene.control.Alert.AlertType;
import java.util.Arrays;
import javafx.scene.control.TextArea;
import javafx.scene.Node;
import javafx.scene.Parent;
import javafx.geometry.Pos;
import javafx.scene.layout.TilePane;
import java.util.Set;
import java.util.HashSet;
import java.util.Random;
import javafx.event.ActionEvent;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;

/**
 * Represents an iTunes Gallery App.
 */
public class GalleryApp extends Application {

    /** HTTP client. */
    public static final HttpClient HTTP_CLIENT = HttpClient.newBuilder()
        .version(HttpClient.Version.HTTP_2)           // uses HTTP protocol version 2 where possible
        .followRedirects(HttpClient.Redirect.NORMAL)  // always redirects, except from HTTPS to HTTP
        .build();                                     // builds and returns a HttpClient object

    /** Google {@code Gson} object for parsing JSON-formatted strings. */
    public static Gson GSON = new GsonBuilder()
        .setPrettyPrinting()                          // enable nice output when printing
        .create();                                    // builds and returns a Gson object

    private Stage stage;
    private Scene scene;
    private VBox root;
    private HBox searchBar;
    private TilePane content;
    private HBox statusBar;
    private ProgressBar progressBar;
    private Label messageBar;
    private ComboBox<String> mediaType;
    private Button getImagesButton;
    private Button playPauseButton;
    private TextField queryTermField;
    private ImageView[] imageViews;
    private int index = 0;
    private String[] imageURIs;
    private String url;
    private String responseBody;
    private List<String> imageURIsList;
    private List<Image> downloadedImages = new ArrayList<>();
    private boolean isPlaying = false;
    private Timeline timeline;

    /**
     * Constructs a {@code GalleryApp} object}.
     */
    public GalleryApp() {
        this.stage = null;
        this.scene = null;
        this.root = new VBox();
        this.searchBar = new HBox();
        this.statusBar = new HBox();
        this.content = new TilePane();
        this.progressBar = new ProgressBar(0.0);
        this.messageBar = new Label("Type in a term, select a media type, then click the button.");
        this.playPauseButton = new Button("Play");
        this.queryTermField = new TextField();
        this.getImagesButton = new Button("Get Images");
        this.imageViews = new ImageView[20];
        this.timeline = new Timeline();
    } // GalleryApp

    /** {@inheritDoc} */
    @Override
    public void init() {
        // feel free to modify this method
        functionality();
        System.out.println("init() called");
    } // init

    /** {@inheritDoc} */
    @Override
    public void start(Stage stage) {
        this.stage = stage;
        this.scene = new Scene(this.root, 500, 470);
        setUp(this.scene);
        this.stage.setOnCloseRequest(event -> Platform.exit());
        this.stage.setTitle("GalleryApp!");
        this.stage.setScene(this.scene);
        this.stage.sizeToScene();
        this.stage.show();
        Platform.runLater(() -> this.stage.setResizable(false));
        //});
    } // start

    /** {@inheritDoc} */
    @Override
    public void stop() {
        // feel free to modify this method
        System.out.println("stop() called");
    } // stop

    /**
     * Sets up the GUI for the application by adding UI components such as
     * the search bar, images, status bar to the scene.
     *
     * @param scene the scene for the application
     */
    private void setUp(Scene scene) {
        messageBar.setStyle("-fx-font-size: 12px;");
        mediaType = new ComboBox<>();
        searchBar.getChildren().addAll(
            playPauseButton,
            new Label("Search:"),
            queryTermField,
            mediaType,
            getImagesButton
        );
        searchBar.setSpacing(5);
        mediaType.getItems().addAll("music", "movie", "podcast", "musicVideo", "audiobook",
             "shortFilm", "tvShow", "software", "ebook", "all");
        mediaType.setValue("music");

        queryTermField.setText("jack johnson");
        queryTermField.setPrefWidth(155);

        VBox fullStatusBar = new VBox();
        fullStatusBar.getChildren().addAll(statusBar);

        statusBar.getChildren().addAll(
            progressBar, new Label("Images provided by iTunes Search API."));
        statusBar.setSpacing(10);
        progressBar.setPrefWidth(200);
        progressBar.setPrefHeight(20);
        fullStatusBar.setAlignment(Pos.BOTTOM_CENTER);

        root.getChildren().addAll(searchBar, messageBar, content, fullStatusBar);

        HBox.setHgrow(content, Priority.ALWAYS);
        HBox.setHgrow(statusBar, Priority.ALWAYS);
        VBox.setVgrow(progressBar, Priority.ALWAYS);
        VBox.setVgrow(fullStatusBar, Priority.ALWAYS);

        HBox.setHgrow(searchBar, Priority.ALWAYS);
        HBox.setHgrow(content, Priority.ALWAYS);
        HBox.setHgrow(statusBar, Priority.ALWAYS);

        Image defaultImage = new Image("file:resources/default.png");
        for (int i = 0; i < imageViews.length; i++) {
            imageViews[i] = new ImageView(defaultImage);
            imageViews[i].setFitWidth(100);
            imageViews[i].setFitHeight(100);
            content.getChildren().add(imageViews[i]);
        }
    }

    /**
     * Puts together the funtionality for components like buttons. This method also handles event
     * actions triggered by the buttons.
     */
    private void functionality() {
        getImagesButton.setOnAction(event -> {
            progressBar.setProgress(0.0);
            playPauseButton.setDisable(true);
            getImagesButton.setDisable(true);
            messageBar.setText("Getting images...");
            playPauseButton.setText("Play");
            String searchTerm = queryTermField.getText();
            searchTerm = searchTerm.toLowerCase();
            if (searchTerm.isEmpty()) {
                showAlert("Enter a search term to get images.");
                return;
            }
            String media = mediaType.getValue();
            String encodedTerm = URLEncoder.encode(searchTerm, StandardCharsets.UTF_8);
            String encodedMedia = URLEncoder.encode(media, StandardCharsets.UTF_8);
            url = "https://itunes.apple.com/search?term=" + encodedTerm +
                "&media=" + encodedMedia + "&limit=200";
            Thread thread = new Thread(() -> {
                HttpRequest request = HttpRequest.newBuilder()
                    .uri(URI.create(url))
                    .build();
                try {
                    HttpResponse<String> response = HTTP_CLIENT.send(
                        request, HttpResponse.BodyHandlers.ofString());
                    Platform.runLater(() -> {
                        if (response.statusCode() != 200) {
                            if (timeline != null && isPlaying) {
                                timeline.stop();
                                isPlaying = false;
                            }
                            showAlert(url, new IOException("(GET \n" + url + ") " +
                                 response.statusCode()));
                        } else {
                            handleResponse(response.body());
                        }
                    });
                } catch (IOException | InterruptedException ie) {
                    showAlert("HTTP request failed for URL: " + url, ie);
                }
            });
            thread.start();
        });
        playPauseButton.setDisable(true);
        playPauseButton.setOnAction(event -> {
            if (playPauseButton.getText().equals("Play")) {
                playPauseButton.setText("Pause");
                Thread startPlayThread = new Thread(() -> {
                    startPlay();
                });
                startPlayThread.start();
            } else {
                playPauseButton.setText("Play");
                Thread stopPlayThread = new Thread(() -> {
                    stopPlay();
                });
                stopPlayThread.start();
            }
        });
    }

    /**
     * Takes in the HTTP response body to get image URLs and download images.
     *
     * @param responseBody the string returned from HTTP request that contains the image URLs.
     */
    private void handleResponse(String responseBody) {
        List<String> imageURIs = new ArrayList<>();

        Gson gson = new Gson();
        ItunesResponse response = gson.fromJson(responseBody, ItunesResponse.class);

        if (response != null && response.results != null) {
            List<String> imageURIsList = extractImageURIs(response.results);

            Thread downloadThread = new Thread(() -> {
                downloadImages(imageURIsList);
            });
            downloadThread.start();
        }
    }

    /**
     * This method downloads images from the URL and updates the UI as well as
     * updating the progress bar. Also handles errors during downloading process.
     *
     * @param imageURIsList the list of URLs for downloading images.
     */
    private void downloadImages(List<String> imageURIsList) {
        if (timeline != null && isPlaying) {
            timeline.stop();
            isPlaying = false;
        }

        this.imageURIsList = imageURIsList;
        int total = imageURIsList.size();
        int count = 0;
        boolean failed = false;
        Throwable t = null;

        downloadedImages.clear();
        if (total < 20) {
            failed = true;
            t = new IllegalArgumentException("0 distinct results found, but 21 or more are needed");
        } else {
            //initial progress shows no progress
            progressBar.setProgress(0.0);
            System.out.println("Image URIs list size: " + total);

            for (String imageURI : imageURIsList) {
                try {
                    Image image = new Image(imageURI, true);
                    downloadedImages.add(image);
                    count++;

                    final double progress = (double) count / total;
                    Platform.runLater(() -> progressBar.setProgress(progress));
                } catch (Throwable e) {
                    showAlert("Error processing images for URL: " + url, e);
                    failed = true;
                    t = e;
                    break;
                }
            }
        }
        final boolean isPlayButtonEnabled = downloadedImages.size() > 0 &&
            downloadedImages.size() < 20;
        final Throwable finalT = t;
        final String currentUrl = this.url;
        final boolean finalFailed = failed;

        Platform.runLater(() -> {
            for (int i = 0; i < downloadedImages.size() && i < imageViews.length; i++) {
                imageViews[i].setImage(downloadedImages.get(i));
            }
            playPauseButton.setDisable(false);
            getImagesButton.setDisable(false);

            if (finalFailed) {
                showAlert(currentUrl, finalT);
            } else {
                messageBar.setText(url);
                progressBar.setProgress(1.0);
            }
        });
    }

    /**
     * Shows an alert box with a specific message.
     *
     * @param message the message displayed in the alert box
     */
    private void showAlert(String message) {
        messageBar.setText("Last attempt to get images failed...");
        progressBar.setProgress(1.0);
        TextArea text = new TextArea(message);
        text.setEditable(false);
        Alert alert = new Alert(AlertType.ERROR);
        alert.setTitle("Error");
        alert.setHeaderText("Error");
        alert.getDialogPane().setContent(text);
        alert.setResizable(true);
        alert.showAndWait();
        playPauseButton.setDisable(false);
        getImagesButton.setDisable(false);
    }

    /**
     * Shows an alert box with specific messages regarding the exception.
     *
     * @param url the url that caused the exception. Includes the search term and media type.
     * @param t the throwable that was caught, causing the alert
     */
    private void showAlert(String url, Throwable t) {
        messageBar.setText("Last attempt to get images failed...");
        progressBar.setProgress(1.0);
        String errorMessage = null;
        if (t != null) {
            errorMessage = String.format("URI: %s\n\nException: %s",
                 url, t.toString());
        }
        TextArea text = new TextArea(errorMessage);
        text.setEditable(false);
        Alert alert = new Alert(AlertType.ERROR);
        alert.setTitle("Error");
        alert.setHeaderText("Error");
        alert.getDialogPane().setContent(text);
        alert.setResizable(true);
        alert.showAndWait();
        playPauseButton.setDisable(false);
        getImagesButton.setDisable(false);
    }

    /**
     * Extracts distinct image URIs for {@link ItunesResult}.
     * The unique URIs are added to a list to avoid duplicate image URIs when displaying images.
     *
     * @param results the array of {@link ItunesResult} objects
     * @return a list of distinct image URIs that were extracted
     */
    private List<String> extractImageURIs(ItunesResult[] results) {
        List<String> imageURIs = new ArrayList<>();
        if (results != null) {
            Set<String> diffURIs = new HashSet<>();
            for (ItunesResult result : results) {
                if (result.artworkUrl100 != null) {
                    diffURIs.add(result.artworkUrl100);
                }
            }
            imageURIs.addAll(diffURIs);
        }
        return imageURIs;
    }

    /**
     * Starts the replacing of images with an interval of 2 seconds.
     */
    private void startPlay() {
        if (!isPlaying) {
            isPlaying = true;
            timeline = new Timeline(new KeyFrame(Duration.seconds(2), event -> {
                replaceRandomImage();
            }));
            timeline.setCycleCount(Timeline.INDEFINITE);
            timeline.play();
        }
    }

    /**
     * Stops the replacing of images and resets the play state.
     */
    private void stopPlay() {
        if (isPlaying) {
            isPlaying = false;
            if (timeline != null) {
                timeline.stop();
            }
        }
    }

    /**
     * Replaces a random image in the app with another random image.
     */
    private void replaceRandomImage() {
        if (downloadedImages.isEmpty()) {
            System.out.println("No downloaded images available to replace");
            return;
        }
        if (!downloadedImages.isEmpty() && imageViews.length > 0) {
            Random rand = new Random();
            int replaceIndex = rand.nextInt(imageViews.length); // index to replace
            Image newImage = downloadedImages.get(rand.nextInt(downloadedImages.size()));

            Platform.runLater(() -> {
                imageViews[replaceIndex].setImage(newImage);
            });
        }
    }

} // GalleryApp
