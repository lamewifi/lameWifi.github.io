import javax.swing.*;
import javax.swing.border.Border;
import javax.swing.event.DocumentEvent;
import javax.swing.event.DocumentListener;
import javax.swing.text.DateFormatter;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.sql.*;
import java.text.SimpleDateFormat;
import java.util.Formatter;

/**
 * Created by Austin on 4/25/2015.
 */
public class GuiLayout extends JFrame implements DocumentListener {

    //Declare private labels
    private boolean changedTF = false;

    private JLabel lblOf = new JLabel("of");
    private JLabel lblStuID = new JLabel("Student ID");
    private JLabel lblCreateStuID = new JLabel("Student ID");
    private JLabel lblFirstName = new JLabel("First Name: ");
    private JLabel lblLastName = new JLabel("Last Name: ");
    private JLabel lblSearchLN = new JLabel("Last Name:");
    private JLabel lblMajor = new JLabel("Major: ");
    private JLabel lblEmail = new JLabel("Email: ");
    private JLabel lblPhoneNum = new JLabel("Phone Number");
    private JLabel lblDOB = new JLabel("Date of Birth: ");
    private JLabel lblAddress = new JLabel("Address: ");
    private JLabel lblZipCode = new JLabel("Zip Code: ");
    private JLabel lblFindZip = new JLabel("Find an entry by Student ID ");
    private JLabel lblFindName = new JLabel("Find an entry by last name ");

    //Decalre Buttons
    private JButton btnPrevious = new JButton("Previous");
    private JButton btnNext = new JButton("Next");
    private JButton btnFindID = new JButton("Find");
    private JButton btnFindName = new JButton("Find");
    private JButton btnBrowseAll = new JButton("Browse All Entries");
    private JButton btnInsertNew = new JButton("Insert New Entry");
    private JButton btnUpdateCurrent = new JButton("Update Current Entry");

    //Decalre TexFields
    private JTextField tfStudentID = new JTextField();



    private JTextField tfFirstName = new JTextField("");
    private JTextField tfLastName = new JTextField("");
    private JTextField tfMajor = new JTextField("");
    private JTextField tfEmail = new JTextField("");
    private JTextField tfPhoneNum = new JTextField("");
    private JTextField tfDOB = new JTextField("");
    private JTextField tfAddress = new JTextField("");
    private JTextField tfZipCode = new JTextField("");
    private JTextField tfFindStuID = new JTextField();
    private JTextField tfFindLstName = new JTextField();
    private JTextField tfPrevious = new JTextField();
    private JTextField tfNext = new JTextField();

    //Create Boarders
    Border Blackline;
    Border blackerLine;

    //Declare panels
    private JPanel pnCreationLables = new JPanel();
    private JPanel pnCreationTF = new JPanel();
    private JPanel pnFindStuID = new JPanel();
    private JPanel pnFindStuLN = new JPanel();
    private JPanel pnEntryBtn = new JPanel();
    private JPanel pnBroseBtn = new JPanel();
    private JPanel pnCreationAll = new JPanel();
    private JPanel bottomThird = new JPanel();


    //Declare Database Connection and related variables
    static final String DATABASE_URL = "jdbc:derby:Education";
    Statement statement = null;

    ResultSet rsAllStudentRecords = null;
    ResultSet resultSet = null;
    int currentStudentIDSelected = 1;
    ResultSetMetaData metaData = null;
    Connection dbConnection = null;
    private PreparedStatement statFindID = null;
    private Statement statBrowseStudents = null;
    private PreparedStatement insertNew = null;
    private PreparedStatement updateRecord = null;
    private int intTotalStuRec = 0;
    private int intCurrentRecSel = 1;
    private ResultSet rsFindStuLastName = null;
    private int intNextRecordRow = 1;
    private int intFirstStuID = 0;
    private PreparedStatement statFindLastName = null;

    public GuiLayout() {
        super("Bank Account GUI");
        declareGuiComponents();
        connectToDatabase();
        //Add panels to Frame.
        add(pnBroseBtn);
        add(pnCreationAll);
        add(bottomThird);



        tfFirstName.getDocument().addDocumentListener(this);
        tfLastName.getDocument().addDocumentListener(this);
        tfMajor.getDocument().addDocumentListener(this);
        tfEmail.getDocument().addDocumentListener(this);
        tfPhoneNum.getDocument().addDocumentListener(this);
        tfDOB.getDocument().addDocumentListener(this);
        tfAddress.getDocument().addDocumentListener(this);
        tfZipCode.getDocument().addDocumentListener(this);


        btnBrowseAll.addActionListener(new ActionListener() {
            @Override
            public void actionPerformed(ActionEvent e) {
                clearFields();
                changedTF = false;
                getAllStudentInfo();
                btnNext.setEnabled(true);
                btnPrevious.setEnabled(true);
                tfNext.setText(Integer.toString(intTotalStuRec));
                tfPrevious.setText(Integer.toString(intCurrentRecSel));
            }
        });

        btnNext.addActionListener(new ActionListener(){

            public void actionPerformed(ActionEvent e) {
                changedTF = false;
                browseStudentRecordsNext();
            }
        });

        btnUpdateCurrent.addActionListener(new ActionListener() {
            public void actionPerformed(ActionEvent e){

                if (changedTF == true){

                    System.out.println(Boolean.valueOf(changedTF));

                    changedTF = false;
                    updateRecord();
                }else {
                    JOptionPane.showMessageDialog(null, "Please make sure you enter new information");
                }

            }
        });

        btnPrevious.addActionListener(new ActionListener(){

            public void actionPerformed(ActionEvent e) {

                browseStudentRecordsPrevious();
            }
        });


        btnInsertNew.addActionListener(new ActionListener(){
            public void actionPerformed(ActionEvent e){
                tfStudentID.setEnabled(true);
                lblCreateStuID.setEnabled(true);
                insertRecord();
            }
        });

        btnFindName.addActionListener(new ActionListener(){
            @Override
            public void actionPerformed(ActionEvent e){
            System.out.println ("Find Name");
                getAllLastNameRecords();
              String stTest = tfFindLstName.getText();

                if (stTest.length() > 25 || stTest.isEmpty()) {
                    JOptionPane.showMessageDialog(null, "Please a valid Last Name");
                }else{
                    tfNext.setText(Integer.toString(intTotalStuRec));
                    tfPrevious.setText(Integer.toString(intCurrentRecSel));

                    getAllLastNameRecords();

                }


            }
        });


        btnFindID.addActionListener(new ActionListener(){
            public void actionPerformed(ActionEvent e){

                String stTest = tfFindStuID.getText();
                try{
                    Integer.parseInt(tfFindStuID.getText());
                    if (stTest.length() < 6 || stTest.isEmpty()) {
                        JOptionPane.showMessageDialog(null, "Please enter a valid student ID");
                    }else{
                        findStudentID();
                    }
                }catch (Exception x){
                    JOptionPane.showMessageDialog(null,"Please make sure the studentID contains only numbers");
                }
            }
        });
    }



    //Gui Components are created here for readiability purposes.
    public void declareGuiComponents(){

        blackerLine = BorderFactory.createBevelBorder(2);
        blackerLine = BorderFactory.createLineBorder(Color.gray);
        // blackerLine = BorderFactory.createStrokeBorder(new BasicStroke(1));

        //Set Layouts for the GUI
        setLayout(new FlowLayout());
        pnCreationLables.setLayout(new GridLayout(9, 1));
        pnCreationTF.setLayout(new GridLayout(9, 1));
        pnFindStuID.setLayout(new GridLayout(1, 3));
        pnFindStuLN.setLayout(new GridLayout(1, 3));
        pnEntryBtn.setLayout(new GridLayout(1, 3));
        pnBroseBtn.setLayout(new GridLayout(1, 5));
        pnCreationAll.setLayout(new GridLayout(1,2));
        bottomThird.setLayout(new GridLayout(5,1));

        //Add Content to relevant panels


        //Manual record browsing
        pnBroseBtn.add(btnPrevious);
        btnPrevious.setEnabled(false);
        pnBroseBtn.add(tfPrevious);
        pnBroseBtn.add(lblOf);
        pnBroseBtn.add(tfNext);
        btnNext.setEnabled(false);
        pnBroseBtn.add(btnNext);

        //Adding TextFields And Labels that will allow manual record creation.

        //Add Content to relevant panels
        pnCreationLables.add(lblCreateStuID);
        lblCreateStuID.setEnabled(false);
        pnCreationLables.add(lblFirstName);
        pnCreationLables.add(lblLastName);
        pnCreationLables.add(lblMajor);
        pnCreationLables.add(lblEmail);
        pnCreationLables.add(lblPhoneNum);
        pnCreationLables.add(lblDOB);
        pnCreationLables.add(lblAddress);
        pnCreationLables.add(lblZipCode);
        //Adding the Textfields to the pnCreationTF Panel
        pnCreationTF.add(tfStudentID);
        tfStudentID.setEnabled(false);
        pnCreationTF.add(tfFirstName);
        pnCreationTF.add(tfLastName);
        pnCreationTF.add(tfMajor);
        pnCreationTF.add(tfEmail);
        pnCreationTF.add(tfPhoneNum);
        pnCreationTF.add(tfDOB);
        pnCreationTF.add(tfAddress);
        pnCreationTF.add(tfZipCode);


        //Adding the creationTF and creationLbl Panels to one panel
        pnCreationAll.add(pnCreationLables);
        pnCreationAll.add(pnCreationTF);


        //Panel to find student by Last Name
        pnFindStuLN.add(lblSearchLN);
        pnFindStuLN.add(tfFindLstName);
        pnFindStuLN.add(btnFindName);

        //Panel to find student by ID
        pnFindStuID.add(lblStuID);
        pnFindStuID.add(tfFindStuID);
        pnFindStuID.add(btnFindID);


        //Last panel to be added
        pnEntryBtn.add(btnBrowseAll);
        pnEntryBtn.add(btnInsertNew);
        pnEntryBtn.add(btnUpdateCurrent);


        //Setting the boarder for the panels
        pnCreationAll.setBorder(blackerLine);
        pnFindStuLN.setBorder(blackerLine);
        pnBroseBtn.setBorder(blackerLine);
        pnFindStuID.setBorder(blackerLine);

        //Adds the browseAll, and Find Student panels
        pnCreationAll.setSize(200,200);
        bottomThird.add(lblFindZip);
        bottomThird.add(pnFindStuID);
        bottomThird.add(lblFindName);
        bottomThird.add(pnFindStuLN);
        bottomThird.add(pnEntryBtn);

    }

    public void connectToDatabase(){
        final String neato = null;
        //Connect to database
        try{
            dbConnection = DriverManager.getConnection(DATABASE_URL,"miller","miller");
            System.out.println("Successfully Connected to Database");
            statBrowseStudents = dbConnection.createStatement( ResultSet.TYPE_SCROLL_INSENSITIVE,
                    ResultSet.CONCUR_UPDATABLE);
            statement = dbConnection.createStatement( ResultSet.TYPE_SCROLL_INSENSITIVE,
                    ResultSet.CONCUR_UPDATABLE);
            insertNew = dbConnection.prepareStatement( "INSERT INTO STUDENT " +
                    "(studentID, firstName, lastName, major, email, phone, birthDate, address, zipCode) " +
                    "Values (?,?,?,?,?,?,?,?,?) "
                     , ResultSet.TYPE_SCROLL_SENSITIVE,  ResultSet.CONCUR_UPDATABLE);
            updateRecord = dbConnection.prepareStatement( "Update Student " +
                    " set firstName = ?, lastName = ?, major = ?, email = ?, phone = ?, birthDate = ?, address = ?, zipCode = ? " +
                    "Where Student.studentId = ?" , ResultSet.TYPE_SCROLL_SENSITIVE,  ResultSet.CONCUR_UPDATABLE);

            statFindID = dbConnection.prepareStatement(   "SELECT * " +
                    "FROM Student "
                    + " Where Student.StudentID = ? ", ResultSet.TYPE_SCROLL_SENSITIVE,  ResultSet.CONCUR_UPDATABLE);
            statFindLastName = dbConnection.prepareStatement(   "SELECT * " +
                    "FROM Student "
                    + " Where lastName = ? ", ResultSet.TYPE_SCROLL_SENSITIVE,  ResultSet.CONCUR_UPDATABLE);

        }catch (SQLException sqle){
            sqle.printStackTrace();
        }
       // getAllStudentInfo();
    }//End connectToDatabasse

    public void updateRecord  (){

                System.out.println(Boolean.valueOf(changedTF));

        try{
            updateRecord.setString(1,tfFirstName.getText());
            updateRecord.setString(2,tfLastName.getText());
            updateRecord.setString(3,tfMajor.getText());
            updateRecord.setString(4,tfEmail.getText());
            updateRecord.setString(5,tfPhoneNum.getText());
            updateRecord.setString(6,tfDOB.getText());
            updateRecord.setString(7,tfAddress.getText());
            updateRecord.setString(8,tfZipCode.getText());
             updateRecord.setString(9,tfStudentID.getText());
            System.out.println();
            updateRecord.addBatch();
            updateRecord.executeBatch();


        }catch (SQLException sqle){
            JOptionPane.showMessageDialog(null,"Error updating student record, please check all fields");
        }




    }
    public void insertRecord(){

        try{
            insertNew.setString(1, tfStudentID.getText());
            insertNew.setString(2,tfFirstName.getText());
            insertNew.setString(3,tfLastName.getText());
            insertNew.setString(4,tfMajor.getText());
            insertNew.setString(5,tfEmail.getText());
            insertNew.setString(6,tfPhoneNum.getText());
            insertNew.setString(7,tfDOB.getText());
            insertNew.setString(8,tfAddress.getText());
            insertNew.setString(9,tfZipCode.getText());

            insertNew.addBatch();
            insertNew.executeBatch();
        }catch (SQLException sqle){
            JOptionPane.showMessageDialog(null,"Error inserting new student, please check all fields");

        }

    }

    public void clearFields(){
        intTotalStuRec = 0;
        intNextRecordRow = 1;
        intCurrentRecSel = 1;

        btnNext.setEnabled(false);
        btnPrevious.setEnabled(false);

        tfStudentID.setText("");
        tfFirstName.setText("");
        tfLastName.setText("");
        tfMajor.setText("");
        tfEmail.setText("");
        tfPhoneNum.setText("");
        tfDOB.setText("");
        tfAddress.setText("");
        tfZipCode.setText("");
        return;
    }



    public void getAllStudentInfo(){


        intTotalStuRec = 0;
        try{
            rsAllStudentRecords = statement.executeQuery("Select * " + " From Student ");
            metaData = rsAllStudentRecords.getMetaData();

            while(rsAllStudentRecords.next()){
                intTotalStuRec++;
            }
            fillWithBrowseAllContent(rsAllStudentRecords);
        }catch (SQLException sqle){
        }


    }
    public void getAllLastNameRecords(){

        intTotalStuRec = 0;
        String stringLastName = tfFindLstName.getText();
        try{
           statFindLastName.setString(1,stringLastName);
            rsAllStudentRecords = statFindLastName.executeQuery();

            while(rsAllStudentRecords.next()){
                intTotalStuRec++;
            }

            if (intTotalStuRec > 1){
                btnNext.setEnabled(true);
                btnPrevious.setEnabled(true);
            }
            else {
                JOptionPane.showMessageDialog(null,"No Student records found");
            }

            fillWithBrowseAllContent(rsAllStudentRecords);

        }catch (SQLException sqle){
        }


    }



    public void browseStudentRecordsNext(){


        if (intNextRecordRow < (intTotalStuRec )){
            intNextRecordRow++;
            btnPrevious.setEnabled(true);
        }else if(intNextRecordRow >= intTotalStuRec ){
            btnNext.setEnabled(false);
            JOptionPane.showMessageDialog(null,"There are no more records");

        }
        fillWithBrowseAllContent(rsAllStudentRecords);
        currentStudentIDSelected = intNextRecordRow;
        tfPrevious.setText(Integer.toString(currentStudentIDSelected));


    }

    public void browseStudentRecordsPrevious(){
        System.out.println("intNextRecordRow Before: " + intNextRecordRow);
        if (intNextRecordRow > 1){
            intNextRecordRow--;

            tfPrevious.setText(Integer.toString(intNextRecordRow));
            btnNext.setEnabled(true);
            System.out.println("intNextRecordRow: " + intNextRecordRow);
        }else if(intNextRecordRow == 1){
            btnPrevious.setEnabled(false);
            currentStudentIDSelected = 1;
            tfPrevious.setText(Integer.toString(1));
            JOptionPane.showMessageDialog(null,"There are no more records");

        }




        fillWithBrowseAllContent(rsAllStudentRecords);

    }

    public void findStudentID() {

        String enteredID = tfFindStuID.getText();
        try{
            statFindID.setString(1,enteredID);
            resultSet = statFindID.executeQuery();
            if (intTotalStuRec >= 0) {
                intNextRecordRow = 1;
                fillWithRS(resultSet);
            }
            else  {
                JOptionPane.showMessageDialog(null,"No Student Records found.");
            }
        }catch (SQLException sqle){
            sqle.printStackTrace();
        }
    }//End findStudentID



    //Method to fill in the textFields with the information from
    //The search by student ID or Last name
    public void fillWithRS(ResultSet rs){

        try {
            while (rs.next()) {
                currentStudentIDSelected = Integer.parseInt(rs.getString(1));
                tfStudentID.setText(rs.getString(1));
                tfFirstName.setText(rs.getString(2));
                tfLastName.setText(rs.getString(3));
                tfMajor.setText(rs.getString(4));
                tfEmail.setText(rs.getString(5));
                tfPhoneNum.setText(rs.getString(6));
                tfDOB.setText(rs.getString(7));
                tfAddress.setText(rs.getString(8));
                tfZipCode.setText(rs.getString(9));
            }
        }catch (SQLException sqle){

        }
        changedTF = false;

    }

    public void fillWithBrowseAllContent(ResultSet rs){

        if (intNextRecordRow <= 0){
            intNextRecordRow =0;
        }


        try {
            rs.absolute(intNextRecordRow);
            currentStudentIDSelected = Integer.parseInt(rs.getString(1));
            tfStudentID.setText(rs.getString(1));
            tfFirstName.setText(rs.getString(2));
            tfLastName.setText(rs.getString(3));
            tfMajor.setText(rs.getString(4));
            tfEmail.setText(rs.getString(5));
            tfPhoneNum.setText(rs.getString(6));
            tfDOB.setText(rs.getString(7));
            tfAddress.setText(rs.getString(8));
            tfZipCode.setText(rs.getString(9));

        }catch (SQLException sqle){

        }

        changedTF = false;
    }


    public void insertUpdate(DocumentEvent e) {

        System.out.println(Boolean.valueOf(changedTF));
        changedTF = true;

    }


    public void removeUpdate(DocumentEvent e) {

        System.out.println(Boolean.valueOf(changedTF));
        changedTF = true;

    }

    public void changedUpdate(DocumentEvent e) {

        System.out.println(Boolean.valueOf(changedTF));
        changedTF = true;

    }
}
