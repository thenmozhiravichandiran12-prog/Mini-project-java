import java.awt.*;
import java.awt.event.*;
import javax.swing.*;
import javax.swing.border.*;

public class FlightBooking extends JFrame implements ActionListener, Runnable {
    
    private JLabel l1, l2, l3, l4, titleLabel, welcomeLabel;
    private JTextField t1, t2, t3;
    private JButton b1, b2;
    private JPanel mainPanel;
    private Timer animationTimer;
    private int planeX = -100, planeY = 120;
    private boolean showWelcome = true;
    private long welcomeStartTime;
    
    public FlightBooking() {
        setTitle("Flight Booking System - AirVista Airlines");
        setSize(750, 650);  // Increased height for welcome
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLocationRelativeTo(null);
        
        mainPanel = new JPanel() {
            @Override
            protected void paintComponent(Graphics g) {
                Graphics2D g2d = (Graphics2D) g;
                g2d.setRenderingHint(RenderingHints.KEY_RENDERING, RenderingHints.VALUE_RENDER_QUALITY);
                
                // Sky gradient background
                GradientPaint skyGradient = new GradientPaint(0, 0, new Color(135, 206, 250), 0, getHeight(), 
                    new Color(70, 130, 180));
                g2d.setPaint(skyGradient);
                g2d.fillRect(0, 0, getWidth(), getHeight());
                
                // Sun
                g2d.setColor(new Color(255, 223, 0));
                g2d.fillOval(650, 30, 60, 60);
                
                // Moving clouds
                g2d.setColor(new Color(255, 255, 255, 200));
                drawCloud(g2d, 100 + (int)(System.currentTimeMillis() * 0.02) % 300, 40, 70);
                drawCloud(g2d, 400 + (int)(System.currentTimeMillis() * 0.015) % 400, 60, 90);
                drawCloud(g2d, 200 + (int)(System.currentTimeMillis() * 0.025) % 250, 90, 60);
                
                // SLIGHTLY FLYING SMALL PLANE (always visible)
                drawSmallAirplane(g2d, 50 + (int)(System.currentTimeMillis() * 0.03) % 200, 80);
                
                // MEDIUM ANIMATED AIRPLANE (after booking)
                if (planeX > -80) {
                    // Airplane trail
                    g2d.setColor(new Color(255, 255, 255, 100));
                    g2d.setStroke(new BasicStroke(5));
                    for (int i = 0; i < 10; i++) {
                        g2d.drawLine(planeX - i*12, planeY + 4, planeX - i*35, planeY + 2);
                    }
                    // Medium airplane
                    drawMediumAirplane(g2d, planeX, planeY);
                }
            }
        };
        
        mainPanel.setOpaque(false);
        mainPanel.setLayout(new GridBagLayout());
        GridBagConstraints gbc = new GridBagConstraints();
        gbc.insets = new Insets(15, 25, 15, 25);
        gbc.fill = GridBagConstraints.HORIZONTAL;
        
        // WELCOME LABEL - FIRST LINE (BLACK & VISIBLE)
        welcomeLabel = new JLabel("Welcome to AirVista Airlines! Book your dream flight now!", JLabel.CENTER);
        welcomeLabel.setFont(new Font("Arial", Font.BOLD, 22));
        welcomeLabel.setForeground(Color.BLACK);
        welcomeLabel.setOpaque(true);
        welcomeLabel.setBackground(new Color(255, 255, 255, 220));
        welcomeLabel.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(Color.BLUE, 3),
            BorderFactory.createEmptyBorder(10, 20, 10, 20)));
        
        // Title Label
        titleLabel = new JLabel("AIRVISTA AIRLINES - Book Your Flight", JLabel.CENTER);
        titleLabel.setFont(new Font("Arial", Font.BOLD, 20));
        titleLabel.setForeground(Color.BLACK);
        titleLabel.setOpaque(false);
        
        // Form labels (BLACK & VISIBLE)
        l1 = new JLabel("Passenger Name:", JLabel.CENTER);
        l2 = new JLabel("Source City:", JLabel.CENTER);
        l3 = new JLabel("Destination:", JLabel.CENTER);
        l4 = new JLabel("Booking Details:", JLabel.CENTER);
        
        styleFormLabel(l1); styleFormLabel(l2); styleFormLabel(l3); styleFormLabel(l4);
        
        t1 = styleTextField();
        t2 = styleTextField();
        t3 = styleTextField();
        
        b1 = new JButton("BOOK TICKET");
        b2 = new JButton("CLEAR FORM");
        styleButton(b1, new Color(34, 139, 34), Color.WHITE);
        styleButton(b2, new Color(255, 69, 0), Color.WHITE);
        
        // Add components - WELCOME FIRST!
        gbc.gridx = 0; gbc.gridy = 0; gbc.gridwidth = 2;
        mainPanel.add(welcomeLabel, gbc);  // FIRST LINE - WELCOME
        
        gbc.gridy = 1;
        mainPanel.add(titleLabel, gbc);    // SECOND LINE - TITLE
        
        gbc.gridy = 2; gbc.gridwidth = 1;
        mainPanel.add(l1, gbc);            // THIRD LINE - Passenger Name
        gbc.gridx = 1;
        mainPanel.add(t1, gbc);
        
        gbc.gridx = 0; gbc.gridy = 3;
        mainPanel.add(l2, gbc);
        gbc.gridx = 1;
        mainPanel.add(t2, gbc);
        
        gbc.gridx = 0; gbc.gridy = 4;
        mainPanel.add(l3, gbc);
        gbc.gridx = 1;
        mainPanel.add(t3, gbc);
        
        gbc.gridx = 0; gbc.gridy = 5; gbc.gridwidth = 2;
        mainPanel.add(b1, gbc);
        gbc.gridy = 6;
        mainPanel.add(b2, gbc);
        
        gbc.gridy = 7;
        l4.setPreferredSize(new Dimension(600, 130));
        l4.setBorder(BorderFactory.createTitledBorder(BorderFactory.createLineBorder(Color.BLUE, 3), 
            "Ticket Information", TitledBorder.CENTER, TitledBorder.TOP));
        mainPanel.add(l4, gbc);
        
        b1.addActionListener(this);
        b2.addActionListener(this);
        
        add(mainPanel);
        
        // Auto-hide welcome after 4 seconds
        welcomeStartTime = System.currentTimeMillis();
        animationTimer = new Timer(30, e -> {
            repaint();
            // Hide welcome label after 4 seconds
            if (showWelcome && System.currentTimeMillis() - welcomeStartTime > 4000) {
                showWelcome = false;
                mainPanel.remove(welcomeLabel);
                mainPanel.revalidate();
                mainPanel.repaint();
            }
        });
        animationTimer.start();
        
        setVisible(true);
    }
    
    private void drawSmallAirplane(Graphics2D g2d, int x, int y) {
        // Small slightly flying plane
        g2d.setColor(Color.LIGHT_GRAY);
        g2d.fillRect(x, y-3, 40, 6);
        g2d.setColor(Color.GRAY);
        g2d.fillPolygon(new int[]{x+10, x+20, x+10}, new int[]{y-8, y-3, y+2}, 3);
        g2d.setColor(Color.CYAN);
        g2d.fillRect(x+8, y-1, 4, 3);
        g2d.fillRect(x+20, y-1, 4, 3);
    }
    
    private void drawMediumAirplane(Graphics2D g2d, int x, int y) {
        // Medium airplane body
        g2d.setColor(Color.LIGHT_GRAY);
        g2d.fillRoundRect(x, y-8, 100, 16, 8, 8);
        
        // Wings
        g2d.setColor(Color.GRAY);
        g2d.fillPolygon(new int[]{x+25, x+50, x+25}, new int[]{y-20, y-8, y+4}, 3);
        g2d.fillPolygon(new int[]{x+75, x+50, x+75}, new int[]{y-20, y-8, y+4}, 3);
        
        // Tail
        g2d.setColor(Color.DARK_GRAY);
        g2d.fillPolygon(new int[]{x-8, x+8, x}, new int[]{y-12, y-12, y+6}, 3);
        
        // Windows
        g2d.setColor(Color.CYAN);
        for (int i = 0; i < 5; i++) {
            g2d.fillRect(x + 20 + i*16, y-5, 10, 8);
        }
    }
    
    private void drawCloud(Graphics2D g2d, int x, int y, int size) {
        g2d.fillOval(x, y, size, size/2);
        g2d.fillOval(x + size/4, y - size/4, (int)(size * 0.67), (int)(size * 0.67));
        g2d.fillOval(x + size/2, y, (int)(size * 0.67), (int)(size * 0.67));
    }
    
    private void styleFormLabel(JLabel label) {
        label.setFont(new Font("Arial", Font.BOLD, 16));
        label.setForeground(Color.BLACK);
        label.setOpaque(false);
        label.setBorder(BorderFactory.createEmptyBorder(12, 12, 12, 12));
    }
    
    private JTextField styleTextField() {
        JTextField tf = new JTextField(18);
        tf.setFont(new Font("Arial", Font.PLAIN, 15));
        tf.setBorder(BorderFactory.createCompoundBorder(
            BorderFactory.createLineBorder(Color.BLACK, 2),
            BorderFactory.createEmptyBorder(10, 12, 10, 12)));
        tf.setBackground(Color.WHITE);
        return tf;
    }
    
    private void styleButton(JButton button, Color bgColor, Color fgColor) {
        button.setFont(new Font("Arial", Font.BOLD, 15));
        button.setBackground(bgColor);
        button.setForeground(fgColor);
        button.setFocusPainted(false);
        button.setBorder(BorderFactory.createRaisedBevelBorder());
        button.setPreferredSize(new Dimension(240, 50));
    }
    
    public void actionPerformed(ActionEvent e) {
        if (e.getSource() == b1) {
            bookTicket();
        } else {
            clearForm();
        }
    }
    
    private void bookTicket() {
        String name = t1.getText().trim();
        String source = t2.getText().trim().toUpperCase();
        String dest = t3.getText().trim().toUpperCase();
        
        if (name.isEmpty() || source.isEmpty() || dest.isEmpty()) {
            l4.setText("<html><center><font color='red' size='+2'>ERROR: All fields are required!</font></center></html>");
            l4.setForeground(Color.RED);
            return;
        }
        
        if (source.equals(dest)) {
            l4.setText("<html><center><font color='red' size='+2'>ERROR: Source and Destination cannot be same!</font></center></html>");
            l4.setForeground(Color.RED);
            return;
        }
        
        String flightNo = "AI" + String.format("%04d", (int)(Math.random() * 9999));
        String bookingId = "BK" + String.format("%06d", (int)(Math.random() * 999999));
        
        String details = "<html><center>" +
                        "<font size='+3' color='green'>TICKET CONFIRMED!</font><br><br>" +
                        "Passenger: <font color='black' size='+2'><b>" + name + "</b></font><br>" +
                        "Flight: <font color='black' size='+2'><b>" + source + " --> " + dest + "</b></font><br>" +
                        "Flight No: <font color='black' size='+2'><b>" + flightNo + "</b></font><br>" +
                        "Booking ID: <font color='black' size='+2'><b>" + bookingId + "</b></font><br>" +
                        "Seat: <font color='black' size='+2'><b>12A</b></font> | <font color='green' size='+2'><b>CONFIRMED</b></font><br><br>" +
                        "<font color='blue' size='+1'>Happy Journey! Safe Travels ✈</font></center></html>";
        
        l4.setText(details);
        l4.setForeground(Color.BLACK);
        
        // Start medium airplane animation
        planeX = -100;
        new Thread(this).start();
    }
    
    @Override
    public void run() {
        while (planeX < getWidth() + 100) {
            planeX += 10;
            try {
                Thread.sleep(40);
            } catch (InterruptedException e) {}
            repaint();
        }
    }
    
    private void clearForm() {
        t1.setText(""); t2.setText(""); t3.setText("");
        l4.setText("<html><center><font color='black' size='+1'>Ready to book your flight! Enter details above.</font></center></html>");
        l4.setForeground(Color.BLACK);
        planeX = -100;
    }
    
    public static void main(String[] args) {
        try {
            UIManager.setLookAndFeel(UIManager.getSystemLookAndFeelClassName());
        } catch (Exception e) {
            e.printStackTrace();
        }
        SwingUtilities.invokeLater(() -> new FlightBooking());
    }
}
