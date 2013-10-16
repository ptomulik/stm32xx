#
# Copyright (c) 2013 by Pawel Tomulik
# 
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
# 
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
# 
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE

#
# This SConscript may be used to build unit-tests for stm32xx library.
#

Import(['env', 'options'])

import re
import SCons.Errors

local_opts =  [ 
    'CMSIS_BASEDIR',
    'MCU_CORE',
    'MCU_FAMILY',
    'MCU_TARGET',
    'SCONSCRIPT_TARGET',
    'STDPERIPH_BASEDIR',
]

# 
# Supported MCU targets
#
mcu_targets = [
    'STM32F10X_CL',
    'STM32F10X_LD',
    'STM32F10X_MD',
    'STM32F10X_HD',
    'STM32F10X_XL',
    'STM32F10X_LD_VL',
    'STM32F10X_MD_VL',
    'STM32F10X_HD_VL',

    # STM32F4xx
#    'STM32F4XX',
#    'STM32F40XX',
#    'STM32F427X',
]

#
# Mapping the supported MCU targets to their families
#
mcu_family_dict = {

    # STM32F10x
    'STM32F10X_CL'      : 'STM32F10X',
    'STM32F10X_LD'      : 'STM32F10X',
    'STM32F10X_MD'      : 'STM32F10X',
    'STM32F10X_HD'      : 'STM32F10X',
    'STM32F10X_XL'      : 'STM32F10X',
    'STM32F10X_LD_VL'   : 'STM32F10X',
    'STM32F10X_MD_VL'   : 'STM32F10X',
    'STM32F10X_HD_VL'   : 'STM32F10X',

    # STM32F4xx
    'STM32F4XX'         : 'STM32F4XX',
    'STM32F40XX'        : 'STM32F4XX',
    'STM32F427X'        : 'STM32F4XX',
}

#
# Mapping MCU family to appropriate cortex core
#
mcu_core_dict = {
    'STM32F10X' : 'cortex-m3',
    'STM32F4XX' : 'cortex-m4',
}


#
# Mapping MCU family to appropriate StdPeriph library.
#
stdperiph_dict = {
    'STM32F10X' : 'STM32F10x_StdPeriph_Driver',
    'STM32F4XX' : 'STM32F4xx_StdPeriph_Driver',
}

#
# Mapping MCU family to appropriate CMSIS directory.
#
cmsis_dict = {
    'STM32F10X' : 'CMSIS/Device/ST/STM32F10x',
    'STM32F4XX' : 'CMSIS/Device/ST/STM32F4xx',
}


#
# Split 'options' into environment overrides and local options.
#
ovrr = dict([(k,v) for k,v in options.iteritems() if k not in local_opts])
opts = dict([(k,v) for k,v in options.iteritems() if k in local_opts])

#
# SCONSCRIPT_TARGET
#
sconscript_target = opts.get('SCONSCRIPT_TARGET', 'lib')

#
# MCU_TARGET
#
try:             mcu_target = opts['MCU_TARGET']
except KeyError: raise SCons.Errors.UserError('You must specify MCU_TARGET')
if not mcu_target.upper() in mcu_targets:
    raise SCons.Errors.UserError('unsupported MCU_TARGET: %s' % mcu_target)
mcu_target = mcu_target.upper()

#
# MCU_FAMILY
#
mcu_family = opts.get('MCU_FAMILY', mcu_family_dict[mcu_target])
mcu_family = mcu_family.upper()

#
# STDPERIPH_BASEDIR and STDPERIPH_DIR
#
try:             stdperiph_basedir = opts['STDPERIPH_BASEDIR']
except KeyError: raise SCons.Errors.UserError('You must specify STDPERIPH_BASEDIR')
stdperiph_dir = '%s/%s' % (stdperiph_basedir, stdperiph_dict[mcu_family])
#
# CMSIS_BASEDIR 
#
try:             cmsis_basedir = opts['CMSIS_BASEDIR']
except KeyError: raise SCons.Errors.UserError('You must specify CMSIS_BASEDIR')

#
# MCU_CORE
#
mcu_core = opts.get('MCU_CORE', mcu_core_dict[mcu_family])
mcu_core = mcu_core.lower()
if sconscript_target != 'unit-test':
    if mcu_core:    mcu_flags = ['-mcpu=%s' % mcu_core, '-mthumb']
    else:           mcu_flags = []
else:
    mcu_flags = []

#
# CPPDEFINES
#
cppdefs = ovrr.get('CPPDEFINES', env.get('CPPDEFINES',[]))
cppdefs = cppdefs[:] # make a copy
if 'USE_STDPERIPH_DRIVER' not in cppdefs: cppdefs.append('USE_STDPERIPH_DRIVER')
if mcu_target not in cppdefs:             cppdefs.append(mcu_target)
ovrr['CPPDEFINES'] = cppdefs

#
# CPPPATH
#
subdirs = ['inc', 'tpl', '%s/Include' % cmsis_dict[mcu_family] ]
cpppath = ovrr.get('CPPPATH', env.get('CPPPATH',[]))
cpppath = cpppath[:] # make a copy
cpppath += ['%s/%s' % (stdperiph_dir, s) for s in subdirs ]
cpppath += ['%s/CMSIS/Include' % cmsis_basedir]
cpppath += ['src']
ovrr['CPPPATH'] = cpppath

#
# CFLAGS
#
cflags = ovrr.get('CFLAGS', env.get('CFLAGS', []))
cflags = cflags[:] # make a copy
cflags += mcu_flags
cflags += ["-std=c99", "-Wa,-adhlns='${TARGET}.lst'"]
ovrr['CFLAGS'] = cflags

#
# CXXFLAGS
#
cxxflags = ovrr.get('CXXFLAGS', env.get('CXXFLAGS', []))[:]
cxxflags = cxxflags[:] # make a copy
cxxflags += mcu_flags
cxxflags += ["-std=c++11", "-Wa,-adhlns='${TARGET}.lst'"]
ovrr['CXXFLAGS'] = cxxflags

#
# LINKFLAGS
#
linkflags = ovrr.get('LINKFLAGS', env.get('LINKFLAGS', []))[:]
linkflags = linkflags[:] # make a copy
linkflags += mcu_flags
ovrr['LINKFLAGS'] = linkflags

#
# LIBS
#
libs = ovrr.get('LIBS', env.get('LIBS', []))
libs = libs[:] # make a copy
ovrr['LIBS'] = libs


if sconscript_target == 'lib':
    #
    # SOURCES
    #
    sources = [ env.Glob('src/stm32xx/*.cpp'), env.Glob('src/stm32xx/*.c') ]
    sources = Flatten(sources)
    #
    # LIBRARY NAME
    #
    libname = "stm32xx_%s" % mcu_target.lower()
    #
    # BUILD THE LIBRARY
    #
    target = env.Library(libname, sources, **ovrr)
elif sconscript_target == 'unit-test':
    #
    # SOURCES
    #
    sources = [ env.File('test/unit/run_tests.cpp'),
                env.Glob('test/unit/stm32xx/*_test.cpp'),
                env.Glob('src/stm32xx/*.cpp'),
                env.Glob('src/stm32xx/*.c'),
                'test/unit/gtest/src/gtest-all.cc']
    sources = Flatten(sources)
    #
    # TEST RUNNER NAME
    #
    progname = "run_tests" 
    #
    # BUILD THE TEST RUNNER
    #
    # FIXME: this shouldn't be hardcoded; also this should be arm compiler
    #        and we should have the unit-tests compiled for and run on 
    #        target boards
    ovrr2 = ovrr.copy()
    #
    # CPPPATH
    #
    ovrr2['CPPPATH'] += ['test/unit/gtest/include','test/unit/gtest']
    #
    # CFLAGS
    #
    ovrr2['CFLAGS'] += ['-pthread', '-Wno-missing-field-initializers']
    #
    # CFLAGS
    #
    ovrr2['CXXFLAGS'] += ['-pthread', '-Wno-missing-field-initializers']
    #
    # LIBS
    #
    ovrr2['LIBS'] += ['CppUTest','pthread']
    ovrr2.update({
        'CXX'  : 'g++',
        'CC'   : 'gcc',
        'LINK' : 'g++',
        'AR'   : 'ar',
    })
    target = env.Program(progname, sources, **ovrr2)
else:
    msg = 'Unsupported SCONSCRIPT_TARGET: %s' % sconscript_target
    raise SCons.Errors.UserError(msg)

#
# SIDE EFFECTS
#
for tgt in target:
  p = tgt.get_path()
  if re.match(r'\.o$', p):
    env.SideEffect(re.sub(r'\.o$', '.o.lst', p), tgt)


#
# RETURN WHAT WAS BUILT
#
Return('target')

# Local Variables:
# # tab-width:4
# # indent-tabs-mode:nil
# # End:
# vim: set syntax=scons expandtab tabstop=4 shiftwidth=4:
