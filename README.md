package com.freecharts.demo;


import java.awt.BasicStroke;
import java.awt.BorderLayout;
import java.awt.Color;
import java.awt.Font;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.FileInputStream;
import java.io.IOException;
import java.io.InputStreamReader;
import java.text.DateFormat;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.ArrayList;
import java.util.Calendar;
import java.util.Date;
import java.util.StringTokenizer;

import javax.swing.JButton;
import javax.swing.JFrame;
import javax.swing.JPanel;

import org.jfree.chart.ChartPanel;
import org.jfree.chart.JFreeChart;
import org.jfree.chart.axis.DateAxis;
import org.jfree.chart.axis.NumberAxis;
import org.jfree.chart.plot.XYPlot;
import org.jfree.chart.renderer.xy.CandlestickRenderer;
import org.jfree.chart.renderer.xy.HighLowRenderer;
import org.jfree.chart.renderer.xy.XYLineAndShapeRenderer;
import org.jfree.data.time.Day;
import org.jfree.data.time.Minute;
import org.jfree.data.time.Month;
import org.jfree.data.time.Second;
import org.jfree.data.time.TimeSeries;
import org.jfree.data.time.TimeSeriesCollection;
import org.jfree.data.time.ohlc.OHLCSeries;
import org.jfree.data.time.ohlc.OHLCSeriesCollection;
import org.jfree.data.xy.OHLCDataItem;
import org.jfree.data.xy.OHLCDataset;
import org.jfree.data.xy.XYDataset;


/**
* Example to combine Candlestick and Line Charts in same chart
* and also convert candlestick to line.
* plus to update the timeseries.
* 
* @author Maneesh Nanu
*
*/
public class OHLC extends JFrame implements ActionListener
{
   private static final long serialVersionUID = 1L;

   private OHLCDataset dataset;
   public static OHLCSeries ohlcSeries;
   private CandlestickRenderer CandleStickRenderer;
   private DateAxis domainAxis;
   private NumberAxis rangeAxis;
   public static XYPlot plot;
   private JFreeChart jfreechart;
   public ChartPanel chartPanel;
   private JPanel panel;
   private XYLineAndShapeRenderer LineRenderer;
   private HighLowRenderer OHLCRenderer;
   protected static int datasetcnt = 1;
   private JButton btnRen;
   private JButton btnAvg;

   public static ArrayList<Date> Date = new ArrayList<Date>();
   public static ArrayList<Double> Close = new ArrayList<Double>();
   public static ArrayList<Double> Open = new ArrayList<Double>();
   public static ArrayList<Double> High = new ArrayList<Double>();
   public static ArrayList<Double> Low = new ArrayList<Double>();

   public OHLC()
   {
      setBackground(Color.WHITE);
      getContentPane().setLayout(new BorderLayout(0, 0));
      dataset = createDataset();
      JFreeChart chart = createChart(dataset);
      chartPanel = new ChartPanel(chart);
      chartPanel.setMouseZoomable(true);

      panel = new JPanel(new BorderLayout());
      panel.add(chartPanel, BorderLayout.CENTER);
      getContentPane().add(panel);

      chart.setBackgroundPaint(Color.WHITE);
      chartPanel.setBackground(Color.WHITE);
      panel.setBackground(Color.WHITE);

      btnRen = new JButton("renderer");
      panel.add(btnRen, BorderLayout.NORTH);
      btnRen.addActionListener(this);

      btnAvg = new JButton("Average");
      panel.add(btnAvg, BorderLayout.SOUTH);
      btnAvg.addActionListener(this);
      this.setVisible(true);
      this.setSize(400, 400);
      setDefaultCloseOperation(EXIT_ON_CLOSE);
   }

   public OHLCDataset createDataset()
   {

      try
      {
         ohlcSeries = new OHLCSeries("");

         Calendar date[] = new Calendar[Date.size()];

         int i;
         Minute minute;
         Second secondOfDay;
         double open, high, low, close;
         
         try
         {
            for (i = 0; i<Date.size(); i++)
            {
               date[i] = Calendar.getInstance();
               
               date[i].setTimeInMillis(Date.get(i).getTime());
               
               minute = new Minute(date[i].get(Calendar.MINUTE),
                     date[i].get(Calendar.HOUR_OF_DAY),
                     date[i].get(Calendar.DAY_OF_MONTH),
                     date[i].get(Calendar.MONTH) + 1,
                     date[i].get(Calendar.YEAR));

               secondOfDay = new Second(date[i].get(Calendar.SECOND),
                     minute);
               open = Open.get(i);
               high = High.get(i);
               low = Low.get(i);
               close = Close.get(i);
               System.out
                     .println(open + "" + high + "" + low + "" + close);
               ohlcSeries.add(secondOfDay, open, high, low, close);

            }
         } catch (Exception e)
         {
            System.out.println(e.getMessage());
         }
         OHLCSeriesCollection ohlcCollection = new OHLCSeriesCollection();
         ohlcSeries.getItemCount();

         ohlcCollection.addSeries(ohlcSeries);
         return ohlcCollection;
      } catch (Exception e)
      {
         System.out.println("Error: " + e);
      } 
      return null;
   }

   @SuppressWarnings("deprecation")
   private JFreeChart createChart(final OHLCDataset dataset)
   {
      CandleStickRenderer = new CandlestickRenderer();
      CandleStickRenderer.setSeriesStroke(0, new BasicStroke(1.0f,
            BasicStroke.CAP_ROUND, BasicStroke.JOIN_BEVEL));
      CandleStickRenderer.setSeriesPaint(0, Color.black);

      LineRenderer = new XYLineAndShapeRenderer(true, false);
      LineRenderer.setStroke(new BasicStroke(1f, BasicStroke.CAP_BUTT,
            BasicStroke.JOIN_BEVEL));
      LineRenderer.setSeriesPaint(0, Color.BLACK);
      LineRenderer.setSeriesPaint(1, Color.BLUE);
      LineRenderer.setSeriesPaint(2, Color.RED);

      OHLCRenderer = new HighLowRenderer();
      OHLCRenderer.setSeriesStroke(0, new BasicStroke(1.0f,
            BasicStroke.CAP_ROUND, BasicStroke.JOIN_BEVEL));
      OHLCRenderer.setSeriesPaint(0, Color.black);

      domainAxis = new DateAxis();
      domainAxis.setAutoRange(true);
      domainAxis.setTickLabelsVisible(true);
      domainAxis.setAutoTickUnitSelection(true);

      rangeAxis = new NumberAxis();
      rangeAxis.setStandardTickUnits(NumberAxis.createStandardTickUnits());
//      rangeAxis.setFixedAutoRange(50.0);
      rangeAxis.setAutoRange(true);

      plot = new XYPlot(dataset, domainAxis, rangeAxis, LineRenderer);
      plot.setBackgroundPaint(Color.LIGHT_GRAY);
      plot.setDomainGridlinePaint(Color.WHITE);
      plot.setRangeGridlinePaint(Color.WHITE);
      plot.setDomainGridlinesVisible(true);
      plot.setRangeGridlinesVisible(true);

      jfreechart = new JFreeChart("Stock Exchange price (IBM)", new Font(
            "SansSerif", Font.BOLD, 24), plot, false);
      return jfreechart;
   }

   public static void main(String[] args) throws Exception, ParseException
   {
      TAFileFunction();
      new OHLC();
   }
   
   public void generateDate()
   {
      try{
      s1.add(new Month(7, 2002), 132.8);
      }
      catch(Exception e){}
   }

   @Override
   public void actionPerformed(ActionEvent ae)
   {
      XYPlot xyplot = (XYPlot) jfreechart.getPlot();
      if (ae.getSource() == btnAvg)
      {
         XYDataset collec = createMADataset();
         // XYDataset dset = new TimeSeriesCollection();
         xyplot.setDataset(2, collec);
         xyplot.setRenderer(2, LineRenderer);

      } else if (ae.getSource() == btnRen)
      {
         xyplot.setRenderer(CandleStickRenderer);
         try{
            s1.addOrUpdate(new Month(7, 2002), a=a+10);
            }
            catch(Exception e){
//               e.printStackTrace();
               System.out.println(e.getMessage());
            }
      }
   }
   static int a = 143;
   TimeSeries s1;
   @SuppressWarnings("deprecation")
   private TimeSeriesCollection createMADataset()
   {
      s1 = new TimeSeries("Budget", "Year", "$ Million",
            Day.class);
      try
      {
         s1.add(new Day(1, 3, 2008), 181.8);
         s1.add(new Day(2, 3, 2008), 181.8);
         s1.add(new Day(3, 3, 2008), 181.8);
         s1.add(new Day(4, 3, 2008), 181.8);
         s1.add(new Day(5, 3, 2008), 181.8);
         s1.add(new Day(6, 3, 2008), 181.8);
         s1.add(new Day(7, 3, 2008), 181.8);
         s1.add(new Day(8, 3, 2008), 181.8);
         s1.add(new Day(9, 3, 2008), 181.8);
         s1.add(new Day(10, 3, 2008), 181.8);
         s1.add(new Day(11, 3, 2008), 181.8);
         s1.add(new Day(12, 3, 2008), 181.8);
         s1.add(new Day(13, 3, 2008), 181.8);
           
         
      }
         
      catch (Exception exception)
      {
         System.err.println(exception.getMessage());
      }
      TimeSeriesCollection timeseriescollection = new TimeSeriesCollection(s1);
      return timeseriescollection;
   }

   
   public static void TAFileFunction() throws IOException, ParseException{
	   
 	  
 	  FileInputStream inFile = new FileInputStream("market.txt");
 	  BufferedReader in = new BufferedReader(new InputStreamReader(inFile));
       DateFormat df = new SimpleDateFormat("y-M-d");

         String inputLine;
         in.readLine();

         while ((inputLine = in.readLine()) != null) {
                 StringTokenizer st = new StringTokenizer(inputLine, ",");

                 Date date       = df.parse( st.nextToken() );
                 double open     = Double.parseDouble( st.nextToken() );
                 double high     = Double.parseDouble( st.nextToken() );
                 double low      = Double.parseDouble( st.nextToken() );
                 double close    = Double.parseDouble( st.nextToken() );
                 double volume   = Double.parseDouble( st.nextToken() );
                 double adjClose = Double.parseDouble( st.nextToken() );

                 Date.add(date);
                 Open.add(open);
                 High.add(high);
                 Low.add(low);
                 Close.add(close);
         }
         in.close();
         inFile.close();
	   
   }
   
   /*
   public static void TAFunction()
   {
      

      Date.add(new Month(2, 2001));
      Date.add(new Month(3, 2001));
      Date.add(new Month(4, 2001));
      Date.add(new Month(5, 2001));
      Date.add(new Month(6, 2001));
      Date.add(new Month(7, 2001));
      Date.add(new Month(8, 2001));
      Date.add(new Month(9, 2001));
      Date.add(new Month(10, 2001));
      Date.add(new Month(11, 2001));
      Date.add(new Month(12, 2001));
      Date.add(new Month(1, 2002));
      Date.add(new Month(2, 2002));
      Date.add(new Month(3, 2002));
      Date.add(new Month(4, 2002));
      Date.add(new Month(5, 2002));
      Date.add(new Month(6, 2002));
      Date.add(new Month(7, 2002));
      Date.add(new Month(8, 2002));
      Date.add(new Month(9, 2002));
      
      
      Close.add(5.0);
      Close.add(40.0);
      Close.add(52.0);
      Close.add(13.0);
      Close.add(13.0);
      Close.add(14.0);
      Close.add(22.0);
      Close.add(40.0);
      Close.add(30.0);
      Close.add(54.0);
      Close.add(46.0);
      Close.add(41.0);
      Close.add(13.0);
      Close.add(57.0);
      Close.add(74.0);
      Close.add(29.0);
      Close.add(34.0);
      Close.add(46.0);
      Close.add(45.0);
      Close.add(73.0);
      Close.add(64.0);
      Close.add(92.0);

      Open.add(10.0);
      Open.add(50.0);
      Open.add(62.0);
      Open.add(23.0);
      Open.add(23.0);
      Open.add(5.0);
      Open.add(32.0);
      Open.add(45.0);
      Open.add(32.0);
      Open.add(45.0);
      Open.add(67.0);
      Open.add(43.0);
      Open.add(23.0);
      Open.add(67.0);
      Open.add(84.0);
      Open.add(23.0);
      Open.add(34.0);
      Open.add(76.0);
      Open.add(45.0);
      Open.add(73.0);
      Open.add(67.0);
      Open.add(96.0);

      High.add(20.0);
      High.add(60.0);
      High.add(72.0);
      High.add(33.0);
      High.add(33.0);
      High.add(15.0);
      High.add(22.0);
      High.add(55.0);
      High.add(42.0);
      High.add(55.0);
      High.add(77.0);
      High.add(53.0);
      High.add(33.0);
      High.add(77.0);
      High.add(94.0);
      High.add(33.0);
      High.add(44.0);
      High.add(86.0);
      High.add(55.0);
      High.add(83.0);
      High.add(77.0);
      High.add(106.0);

      Low.add(10.0);
      Low.add(50.0);
      Low.add(62.0);
      Low.add(23.0);
      Low.add(23.0);
      Low.add(5.0);
      Low.add(32.0);
      Low.add(45.0);
      Low.add(32.0);
      Low.add(45.0);
      Low.add(67.0);
      Low.add(43.0);
      Low.add(23.0);
      Low.add(67.0);
      Low.add(84.0);
      Low.add(23.0);
      Low.add(34.0);
      Low.add(76.0);
      Low.add(45.0);
      Low.add(73.0);
      Low.add(67.0);
      Low.add(96.0);
   }*/
}
