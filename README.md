
// RemoveBgApp.java import
java.io.; import java.net.HttpURLConnection; import java.net.URL; import java.nio.file.Files; import javax.swing.; import java.awt.; import java.awt.event.;

public class RemoveBgApp { private static final String API_KEY = "YOUR_API_KEY_HERE";

public static void main(String[] args) {
    JFrame frame = new JFrame("Background Remover");
    frame.setSize(400, 300);
    frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
    frame.setLayout(new FlowLayout());

    JButton uploadButton = new JButton("Upload Image");
    JButton processButton = new JButton("Remove Background");
    JLabel statusLabel = new JLabel("Status: Waiting...");

    final File[] selectedFile = new File[1];

    uploadButton.addActionListener(e -> {
        JFileChooser fileChooser = new JFileChooser();
        if (fileChooser.showOpenDialog(null) == JFileChooser.APPROVE_OPTION) {
            selectedFile[0] = fileChooser.getSelectedFile();
            statusLabel.setText("Selected: " + selectedFile[0].getName());
        }
    });

    processButton.addActionListener(e -> {
        if (selectedFile[0] == null) {
            JOptionPane.showMessageDialog(frame, "Please select an image first.");
            return;
        }
        try {
            byte[] imageBytes = Files.readAllBytes(selectedFile[0].toPath());
            HttpURLConnection connection = (HttpURLConnection) new URL("https://api.remove.bg/v1.0/removebg").openConnection();
            connection.setRequestMethod("POST");
            connection.setRequestProperty("X-Api-Key", API_KEY);
            connection.setDoOutput(true);

            String boundary = "----WebKitFormBoundary" + System.currentTimeMillis();
            connection.setRequestProperty("Content-Type", "multipart/form-data; boundary=" + boundary);

            OutputStream output = connection.getOutputStream();
            PrintWriter writer = new PrintWriter(new OutputStreamWriter(output, "UTF-8"), true);

            // Image file part
            writer.append("--" + boundary + "\r\n");
            writer.append("Content-Disposition: form-data; name=\"image_file\"; filename=\"" + selectedFile[0].getName() + "\"\r\n");
            writer.append("Content-Type: " + Files.probeContentType(selectedFile[0].toPath()) + "\r\n\r\n");
            writer.flush();
            output.write(imageBytes);
            output.flush();
            writer.append("\r\n").flush();

            // Size part
            writer.append("--" + boundary + "\r\n");
            writer.append("Content-Disposition: form-data; name=\"size\"\r\n\r\n");
            writer.append("auto\r\n").flush();

            writer.append("--" + boundary + "--\r\n").flush();
            writer.close();

            int responseCode = connection.getResponseCode();
            if (responseCode == 200) {
                InputStream inputStream = connection.getInputStream();
                File outputFile = new File("output.png");
                Files.copy(inputStream, outputFile.toPath(), java.nio.file.StandardCopyOption.REPLACE_EXISTING);
                inputStream.close();
                statusLabel.setText("Image processed. Saved as output.png");
                JOptionPane.showMessageDialog(frame, "Background removed successfully.");
            } else {
                InputStream errorStream = connection.getErrorStream();
                String error = new BufferedReader(new InputStreamReader(errorStream))
                        .lines().reduce("", (acc, line) -> acc + line);
                JOptionPane.showMessageDialog(frame, "Error: " + error);
            }
        } catch (IOException ex) {
            ex.printStackTrace();
            JOptionPane.showMessageDialog(frame, "Exception occurred: " + ex.getMessage());
        }
    });

    frame.add(uploadButton);
    frame.add(processButton);
    frame.add(statusLabel);
    frame.setVisible(true);
}

}


