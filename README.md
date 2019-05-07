# RMI-Remote-Method-Invocation for Telephone Direcory
This repo contains demonstration of Distributed System with RMI - Remote Method Invocation.
##Step I: Define Services in the Interface (IDL)

package phonedir;

import java.rmi.Remote;
import java.rmi.RemoteException;
import java.util.ArrayList;

public interface RMIInterface extends Remote {

    public String addContact(String contactName, String phoneNo) throws RemoteException;

    public ArrayList searchContact(String phoneNo) throws RemoteException;

    public ArrayList viewAllContacts(int i) throws RemoteException;

    public String updateContact(String contactName, String phoneNo) throws RemoteException;

    public String deleteContact(String phoneNo) throws RemoteException;

}

##Implement the Server Side (RMIServer.java)

package phonedir;

import java.rmi.RemoteException;
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;
import java.rmi.server.UnicastRemoteObject;
import java.sql.*;
import java.util.ArrayList;

public class RMIServer extends UnicastRemoteObject implements RMIInterface {

    static final String DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://localhost/phonedir";
    Connection conn = null;
    Statement stmt = null;
    ResultSet rs;
    PreparedStatement prst;
    ResultSetMetaData rsmd;

    public RMIServer() throws RemoteException {

    }

    public String addContact(String contactName, String phoneNo) throws RemoteException {
        String phoneNumber = "", val = "";
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(DB_URL, "root", "");
            stmt = conn.createStatement();
            String sql = "SELECT * FROM contact WHERE phoneno = '" + phoneNo + "'";
            rs = stmt.executeQuery(sql);
            while (rs.next()) {
                phoneNumber = rs.getString("phoneno");
            }

            if (phoneNo.intern().equals(phoneNumber)) {
                val = "0"; //Duplicate
            } else if (!phoneNo.intern().equals(phoneNumber)) {
                stmt = conn.createStatement();
                String sql1 = "INSERT INTO contact(contactname, phoneno) VALUES('" + contactName + "','" + phoneNo + "')";
                stmt.executeUpdate(sql1);
                val = "1"; //Successfully Registered
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return val;
    }

    public ArrayList searchContact(String phoneNo) throws RemoteException {
        ArrayList array = new ArrayList();
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(DB_URL, "root", "");
            stmt = conn.createStatement();
            String sql = "SELECT *FROM contact WHERE phoneno ='" + phoneNo + "'";
            rs = stmt.executeQuery(sql);
            while (rs.next()) {
                for (int i = 0; i < 2; i++) {
                    array.add(rs.getString(i + 1));
                }
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return array;
    }

    public ArrayList viewAllContacts(int i) throws RemoteException {
        ArrayList array = new ArrayList();
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(DB_URL, "root", "");
            stmt = conn.createStatement();
            String sql = "SELECT contactname, phoneno FROM contact ORDER BY contactname DESC";
            stmt.execute(sql);
            rs = stmt.getResultSet();
            while (rs.next()) {
                array.add(rs.getObject(i + 1));
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return array;
    }

    public String updateContact(String contactName, String phoneNo) throws RemoteException {
        String val = "", CPHONE = "";
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(DB_URL, "root", "");
            stmt = conn.createStatement();
            String sql = "SELECT *FROM contact WHERE phoneno = '" + phoneNo + "'";
            rs = stmt.executeQuery(sql);

            while (rs.next()) {
                CPHONE = rs.getString("phoneno");
            }
            if (!phoneNo.intern().equals(CPHONE)) {
                val = "0";
            } else if (phoneNo.intern().equals(CPHONE)) {
                prst = conn.prepareStatement("UPDATE contact SET contactname = '" + contactName + "' WHERE phoneno = '" + phoneNo + "'");
                prst.executeUpdate();
                val = "1";
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return val;
    }

    public String deleteContact(String phoneNo) throws RemoteException {
        String val = "", CPHONE = "";
        try {
            Class.forName(DRIVER);
            conn = DriverManager.getConnection(DB_URL, "root", "");
            stmt = conn.createStatement();
            String sql = "SELECT *FROM contact WHERE phoneno = '" + phoneNo + "'";
            rs = stmt.executeQuery(sql);

            while (rs.next()) {
                CPHONE = rs.getString("phoneno");
            }
            if (!phoneNo.intern().equals(CPHONE)) {
                val = "0";
            }

            else if (phoneNo.intern().equals(CPHONE)) {
                stmt = conn.createStatement();
                String sql2 = "DELETE FROM contact WHERE phoneno = '"+phoneNo+"'";
                stmt.executeUpdate(sql2);
                val = "1";
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return val;
    }

    public static void main(String[] args) {
        try {
            Registry r = LocateRegistry.createRegistry(1010);
            RMIServer rs = new RMIServer();
            r.rebind("x", rs);
            System.out.println("Server Ready!");
        } catch (Exception ex) {
            ex.printStackTrace();
        }

    }
}

##Finally Implement Client Side
##Create Java SE GUI

private void btnRegisterActionPerformed(java.awt.event.ActionEvent evt) {                                            
        // TODO add your handling code here:
        try {
            String contactName = txtName.getText();
            String phoneNo = txtPhoneNo.getText();

            RMIInterface ri = (RMIInterface) Naming.lookup("rmi://localhost:1010/x");
            String res = ri.addContact(contactName, phoneNo);

            if (res.equals("1")) {
                JOptionPane.showMessageDialog(this, "Contact has been Registered!", "Phone Dir: Contact Registered.", JOptionPane.INFORMATION_MESSAGE);
            } else if (res.equals("0")) {
                JOptionPane.showMessageDialog(this, "Contact already Exist!", "Phone Dir: Contact Exist.", JOptionPane.INFORMATION_MESSAGE);
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }                                           

    private void btnSearchActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // TODO add your handling code here:
        ArrayList array = new ArrayList();
        try {
            String phoneNo = txtSearchPhone.getText();
            RMIInterface ri = (RMIInterface) Naming.lookup("rmi://localhost:1010/x");
            for (int i = 0; i < 2; i++) {
                array = ri.searchContact(phoneNo);
            }
            if (!array.isEmpty()) {
                txtName.setText(array.get(0).toString());
                txtPhoneNo.setText(array.get(1).toString());
            } else if (array.isEmpty()) {
                JOptionPane.showMessageDialog(this, "No record found.", "Phone Dire: No Record Found", JOptionPane.INFORMATION_MESSAGE);
            }
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }                                         

    private void btnUpdateActionPerformed(java.awt.event.ActionEvent evt) {                                          
        try {
            String contactName = txtName.getText();
            String phoneNo = txtPhoneNo.getText();
            RMIInterface ri = (RMIInterface) Naming.lookup("rmi://localhost:1010/x");
            String result = ri.updateContact(contactName, phoneNo);

            if (result.equals("1")) {
                JOptionPane.showMessageDialog(this, "Contact Updated Successfully!", "Tele Dir: Contact Updated.", JOptionPane.INFORMATION_MESSAGE);

            } else if (result.equals("0")) {
                JOptionPane.showMessageDialog(this, "Contact Doesn't Exist!", "Tele Dir: No Contact.", JOptionPane.INFORMATION_MESSAGE);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

    }                                         

    private void btnDeleteActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // TODO add your handling code here:
        try {
            String phoneNo = txtPhoneNo.getText();
            
            RMIInterface ri = (RMIInterface) Naming.lookup("rmi://localhost:1010/x");
            String result = ri.deleteContact(phoneNo);

            if (result.equals("1")) {
                JOptionPane.showMessageDialog(this, "Contact Deleted Successfully!", "Tele Dir: Contact Deleted.", JOptionPane.INFORMATION_MESSAGE);

            } else if (result.equals("0")) {
                JOptionPane.showMessageDialog(this, "Contact Doesn't Exist!", "Tele Dir: No Contact.", JOptionPane.INFORMATION_MESSAGE);
            }

        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    
    Step IV: Run the Server Side
    Step V: Run the Client Side GUI

