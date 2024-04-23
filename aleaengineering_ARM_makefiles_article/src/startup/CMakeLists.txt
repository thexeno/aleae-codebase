file(GLOB AleaExample "startup_stm32f103c8tx.s")
add_library(hstartup STATIC ${AleaExample})