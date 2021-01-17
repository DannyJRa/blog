A command line solution:

Export your SVG master.svg to PNG with Inkscape:
# Install on Ubuntu
sudo apt-get install inkscape
# Other systems: make sure Inkscape is in your PATH

inkscape -w 16 -h 16 -e 16.png master.svg
inkscape -w 32 -h 32 -e 32.png master.svg
inkscape -w 48 -h 48 -e 48.png master.svg
Convert the PNG images to ICO with ImageMagick:
# Install on Ubuntu
sudo apt-get install imagemagick

convert 16.png 32.png 48.png icon.ico
Optional - Make sure your ICO contains everything:
$ identify icon.ico
icon.ico[1] ICO 16x16 16x16+0+0 32-bit sRGB 21.2KB 0.000u 0:00.000
icon.ico[0] ICO 32x32 32x32+0+0 32-bit sRGB 21.2KB 0.000u 0:00.000
icon.ico[0] ICO 48x48 48x48+0+0 32-bit sRGB 21.2KB 0.000u 0:00.000


sudo apt-get install imagemagick

convert -density 384 favicon.svg -define icon:auto-resize favicon.ico