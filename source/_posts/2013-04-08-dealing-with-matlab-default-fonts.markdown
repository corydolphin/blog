---
layout: post
title: "Dealing with MATLAb default fonts"
date: 2013-04-08 17:02
comments: true
categories: code
---
I have recently been working heavily with MATLAB for an introduction to Microeletronic Circuits course at Olin College, where we explore the properties and behavior of transistors. The class has a significant lab component, which means I am preparing a large number of different figures using MATLAB. In the process, I have found a number of pain points:
*The MATLAB default fonts are far too small to be legible when embedded in a LaTeX document.
*When calling `saveas` on a figure, the image produced is the same size as whatever the frame of the current figure is.


{% codeblock Getting meta with Matlab -doplot.m %}
function doplot(path_to_mfile,figure_folder_path,figure_name)
    newfigure();
    run(path_to_mfile);
    saveformatfig(figure_folder_path,figure_name);
end
{% endcodeblock %}