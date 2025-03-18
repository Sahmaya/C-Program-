# C-Program-

<h1>C++ Program Assignment</h1>


<h2>Description</h2>
This C++ program was developed for a computer science assignment requiring image analysis. The program processes an image of buttons, identifies damaged and undamaged ones, and visually marks them by overlaying red squares on damaged buttons and green squares on undamaged ones. 
<br />


<h2>This is the original buttons image</h2>
<img width="591" alt="original_buttons" src="https://github.com/user-attachments/assets/75397889-b9a5-488a-9607-cc41161b51fe" />



<h2> Here is the following C++ Program </h2>
[Upl// Sahmaya Anderson-Edwards
// 24012404
// Assignment 3

#include <iostream>
#include <fstream>
#include <cstdlib>
using namespace std;

class pixel_class
{
private:
    int red, green, blue;
    bool exclude; // if true, do not check this pixel
public:
    void loaddata(int v1, int v2, int v3);
    void datatofile(fstream &ppmfile);
    int getR() { return red; }
    int getG() { return green; }
    int getB() { return blue; }
    void setexclude(bool ex) { exclude = ex; }
    bool getexclude() { return exclude; }
};

void loadButtons();
void findConnectedPixels(int x, int y);
void drawBox(int xmin, int xmax, int ymin, int ymax, int R, int G, int B);
void saveImage();

int total, xmin, xmax, ymin, ymax;
int screenx, screeny, maxcolours;
int referencebutton = 0;
pixel_class picture[600][600];

int main()
{
    // Step 1: read in the image from Buttons.ppm
    loadButtons();

    // Step 2: Identify largest button and set it as the reference button
    for (int y = 0; y < screeny; y++)
    {
        for (int x = 0; x < screenx; x++)
        {
            if (!picture[x][y].getexclude() && picture[x][y].getR() > 128)
            {
                total = 0;
                xmin = screenx;
                xmax = 0;
                ymin = screeny;
                ymax = 0;

                // Find all connected pixels (button)
                findConnectedPixels(x, y);

                // Store the size of the largest button found as the reference
                if (total > referencebutton)
                {
                    referencebutton = total;
                }

                // Mark the button as excluded to prevent reprocessing
                for (int i = ymin; i <= ymax; i++)
                {
                    for (int j = xmin; j <= xmax; j++)
                    {
                        if (j >= 0 && j < screenx && i >= 0 && i < screeny)
                        { // Check bounds
                            picture[j][i].setexclude(true);
                        }
                    }
                }
            }
        }
    }

    // Reset exclusion for second round
    for (int y = 0; y < screeny; y++)
    {
        for (int x = 0; x < screenx; x++)
        {
            picture[x][y].setexclude(false);
        }
    }

    // Step 3: Compare all buttons to the reference size and mark as damaged or normal
    for (int y = 0; y < screeny; y++)
    {
        for (int x = 0; x < screenx; x++)
        {
            if (!picture[x][y].getexclude() && picture[x][y].getR() > 128)
            {
                total = 0;
                xmin = screenx;
                xmax = 0;
                ymin = screeny;
                ymax = 0;

                // Find all connected pixels (button)
                findConnectedPixels(x, y);

                // Check if button is damaged by comparing to reference button
                bool isDamaged = total < referencebutton * 0.95;

                // Draw the box around the button
                if (isDamaged)
                {
                    drawBox(xmin, xmax, ymin, ymax, 255, 0, 0); // Red for damaged
                }
                else
                {
                    drawBox(xmin, xmax, ymin, ymax, 0, 255, 0); // Green for normal
                }

                // Mark the button as excluded
                for (int i = ymin; i <= ymax; i++)
                {
                    for (int j = xmin; j <= xmax; j++)
                    {
                        if (j >= 0 && j < screenx && i >= 0 && i < screeny)
                        {
                            picture[j][i].setexclude(true);
                        }
                    }
                }
            }
        }
    }

    // Step 4: Output the final .ppm file
    saveImage();
    return 0;
}

void loadButtons()
{
    // load the picture from Buttons.ppm
    int x, y, R, G, B;
    fstream infile;
    string infilename, line;
    infilename = "Buttons.ppm";
    infile.open(infilename.c_str(), fstream::in);
    if (infile.is_open() == false)
    {
        cout << "ERROR: not able to open " << infilename << endl;
        exit(2);
    }
    getline(infile, line);        // this line is "P3"
    getline(infile, line);        // this line is "# filename"
    infile >> screenx >> screeny; // this line is the size
    infile >> maxcolours;         // this line is 256
    for (y = 0; y < screeny; y++)
    {
        for (x = 0; x < screenx; x++)
        {
            infile >> R >> G >> B;
            picture[x][y].loaddata(R, G, B);
            picture[x][y].setexclude(false);
        }
    }
    infile.close();
}
void findConnectedPixels(int x, int y)
{
    if (x < 0 || x >= screenx || y < 0 || y >= screeny)
    {
        return;
    }
    if (picture[x][y].getexclude() == true)
    {
        return;
    }
    if (picture[x][y].getR() <= 128)
    {
        return;
    }
    total++;
    picture[x][y].setexclude(true);

    if (x < xmin)
        xmin = x;
    if (x > xmax)
        xmax = x;
    if (y < ymin)
        ymin = y;
    if (y > ymax)
        ymax = y;

    findConnectedPixels(x - 1, y);
    findConnectedPixels(x + 1, y);
    findConnectedPixels(x, y - 1);
    findConnectedPixels(x, y + 1);
}

// Function to draw a box
void drawBox(int xmin, int xmax, int ymin, int ymax, int R, int G, int B)
{

    for (int x = xmin; x <= xmax; x++)
    {
        if (ymin >= 0 && ymin < screeny)
            picture[x][ymin].loaddata(R, G, B); // Top
        if (ymax >= 0 && ymax < screeny)
            picture[x][ymax].loaddata(R, G, B); // Bottom
    }

    for (int y = ymin; y <= ymax; y++)
    {
        if (xmin >= 0 && xmin < screenx)
            picture[xmin][y].loaddata(R, G, B); // Left side
        if (xmax >= 0 && xmax < screenx)
            picture[xmax][y].loaddata(R, G, B); // Right side
    }
}

// Save the final image to Output.ppm
void saveImage()
{
    fstream outfile;
    outfile.open("output.ppm", fstream::out);
    if (!outfile.is_open())
    {
        cout << "ERROR: not able to open output.ppm" << endl;
        exit(2);
    }
    outfile << "P3\n"
            << screenx << " " << screeny << "\n"
            << maxcolours << "\n";
    for (int y = 0; y < screeny; y++)
    {
        for (int x = 0; x < screenx; x++)
        {
            picture[x][y].datatofile(outfile);
        }
    }
    cout << "file saved to Output.ppm" << endl;
    outfile.close();
}

//--------------- methods for the pixel_class ------------
void pixel_class::loaddata(int v1, int v2, int v3)
{
    red = v1;
    green = v2;
    blue = v3;
}

void pixel_class::datatofile(fstream &ppmfile)
{
    // write the data for one pixel to the ppm file
    ppmfile << red << " " << green;
    ppmfile << " " << blue << "  ";
}
oading C++Program- Sahmaya Anderson-Edwards .cpp…]()



<h2>The final image output:</h2>
<img width="596" alt="output" src="https://github.com/user-attachments/assets/bc760582-9cac-482b-a89d-b3cce8f70128" />





